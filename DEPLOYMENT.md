# Deployment Guide - Irys Time Capsule

This guide provides comprehensive instructions for deploying the Irys Time Capsule application to production environments.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Smart Contract Deployment](#smart-contract-deployment)
- [Backend Deployment](#backend-deployment)
- [Frontend Build and Integration](#frontend-build-and-integration)
- [Production Server Setup](#production-server-setup)
- [Domain and SSL Configuration](#domain-and-ssl-configuration)
- [Monitoring and Maintenance](#monitoring-and-maintenance)
- [Troubleshooting](#troubleshooting)

## Prerequisites

Before deploying to production, ensure you have:

### Required Accounts and Services
- **Irys Network Access**: Account and private key for Irys blockchain
- **Domain Name**: Registered domain for your application
- **SSL Certificate**: For HTTPS encryption
- **Server Instance**: VPS or cloud server (AWS, DigitalOcean, etc.)

### Required Software
- **Node.js** (v18 or higher)
- **Python** (v3.11 or higher)
- **Git** for version control
- **Nginx** for reverse proxy
- **PM2** for process management
- **Certbot** for SSL certificates

### Required Credentials
- **Private Key**: For smart contract deployment
- **Irys RPC URL**: Network endpoint
- **Server SSH Access**: For deployment automation

## Environment Setup

### 1. Server Preparation

Update your server and install required packages:

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install Python and pip
sudo apt install python3.11 python3.11-venv python3-pip -y

# Install Nginx
sudo apt install nginx -y

# Install PM2 globally
sudo npm install -g pm2

# Install Certbot for SSL
sudo apt install certbot python3-certbot-nginx -y
```

### 2. Environment Variables

Create a production environment file:

```bash
# Create environment file
sudo nano /etc/environment
```

Add the following variables:

```bash
# Irys Network Configuration
IRYS_PRIVATE_KEY=your_private_key_here
IRYS_RPC_URL=https://rpc.irys.xyz
IRYS_CHAIN_ID=1337

# Smart Contract Configuration
CONTRACT_ADDRESS=deployed_contract_address
CONTRACT_OWNER=your_wallet_address

# Application Configuration
FLASK_ENV=production
FLASK_SECRET_KEY=your_secret_key_here
API_BASE_URL=https://yourdomain.com/api

# Database Configuration (if using)
DATABASE_URL=sqlite:///app.db

# Security Configuration
CORS_ORIGINS=https://yourdomain.com
ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com
```

### 3. User and Directory Setup

Create a dedicated user for the application:

```bash
# Create application user
sudo adduser irys-app
sudo usermod -aG sudo irys-app

# Switch to application user
sudo su - irys-app

# Create application directory
mkdir -p /home/irys-app/irys-time-capsule
cd /home/irys-app/irys-time-capsule
```

## Smart Contract Deployment

### 1. Prepare Deployment Environment

Clone the repository and install dependencies:

```bash
git clone https://github.com/your-username/irys-time-capsule.git .
npm install
```

### 2. Configure Hardhat Network

Update `hardhat.config.js` for Irys mainnet:

```javascript
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

module.exports = {
  solidity: {
    version: "0.8.20",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200
      }
    }
  },
  networks: {
    irys: {
      url: process.env.IRYS_RPC_URL,
      accounts: [process.env.IRYS_PRIVATE_KEY],
      chainId: parseInt(process.env.IRYS_CHAIN_ID),
      gasPrice: "auto"
    }
  },
  etherscan: {
    apiKey: {
      irys: process.env.IRYS_API_KEY
    },
    customChains: [
      {
        network: "irys",
        chainId: parseInt(process.env.IRYS_CHAIN_ID),
        urls: {
          apiURL: "https://explorer.irys.xyz/api",
          browserURL: "https://explorer.irys.xyz"
        }
      }
    ]
  }
};
```

### 3. Create Deployment Script

Create `scripts/deploy-production.js`:

```javascript
const { ethers } = require("hardhat");

async function main() {
  console.log("Deploying TimeCapsule contract to Irys mainnet...");

  // Get the contract factory
  const TimeCapsule = await ethers.getContractFactory("TimeCapsule");

  // Deploy the contract
  const timeCapsule = await TimeCapsule.deploy();
  await timeCapsule.waitForDeployment();

  const contractAddress = await timeCapsule.getAddress();
  console.log("TimeCapsule deployed to:", contractAddress);

  // Verify deployment
  const creationFee = await timeCapsule.creationFee();
  console.log("Creation fee:", ethers.formatEther(creationFee), "ETH");

  // Save deployment info
  const deploymentInfo = {
    contractAddress: contractAddress,
    network: "irys",
    deployedAt: new Date().toISOString(),
    creationFee: creationFee.toString()
  };

  require("fs").writeFileSync(
    "deployment-info.json",
    JSON.stringify(deploymentInfo, null, 2)
  );

  console.log("Deployment info saved to deployment-info.json");
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

### 4. Deploy Contract

Execute the deployment:

```bash
# Compile contracts
npx hardhat compile

# Run tests
npx hardhat test

# Deploy to Irys mainnet
npx hardhat run scripts/deploy-production.js --network irys

# Verify contract (optional)
npx hardhat verify --network irys DEPLOYED_CONTRACT_ADDRESS
```

### 5. Update Environment Variables

Update your environment file with the deployed contract address:

```bash
sudo nano /etc/environment
# Update CONTRACT_ADDRESS with the deployed address
```

## Backend Deployment

### 1. Setup Python Environment

```bash
cd /home/irys-app/irys-time-capsule/backend

# Create virtual environment
python3.11 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Install production server
pip install gunicorn
```

### 2. Configure Production Settings

Create `src/config.py`:

```python
import os
from datetime import timedelta

class ProductionConfig:
    SECRET_KEY = os.environ.get('FLASK_SECRET_KEY')
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL', 'sqlite:///app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    
    # CORS Configuration
    CORS_ORIGINS = os.environ.get('CORS_ORIGINS', '').split(',')
    
    # Irys Configuration
    IRYS_PRIVATE_KEY = os.environ.get('IRYS_PRIVATE_KEY')
    IRYS_RPC_URL = os.environ.get('IRYS_RPC_URL')
    CONTRACT_ADDRESS = os.environ.get('CONTRACT_ADDRESS')
    
    # Security
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
    PERMANENT_SESSION_LIFETIME = timedelta(hours=24)
    
    # File Upload
    MAX_CONTENT_LENGTH = 50 * 1024 * 1024  # 50MB max file size
    UPLOAD_FOLDER = '/tmp/uploads'
    
    # Rate Limiting
    RATELIMIT_STORAGE_URL = 'memory://'
    RATELIMIT_DEFAULT = "100 per hour"
```

### 3. Update Main Application

Modify `src/main.py` for production:

```python
import os
import sys
sys.path.insert(0, os.path.dirname(os.path.dirname(__file__)))

from flask import Flask, send_from_directory
from flask_cors import CORS
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address
from src.models.user import db
from src.routes.user import user_bp
from src.routes.capsule import capsule_bp
from src.config import ProductionConfig

def create_app():
    app = Flask(__name__, static_folder=os.path.join(os.path.dirname(__file__), 'static'))
    
    # Load configuration
    app.config.from_object(ProductionConfig)
    
    # Initialize extensions
    CORS(app, origins=app.config['CORS_ORIGINS'])
    
    # Rate limiting
    limiter = Limiter(
        app,
        key_func=get_remote_address,
        default_limits=["100 per hour"]
    )
    
    # Register blueprints
    app.register_blueprint(user_bp, url_prefix='/api')
    app.register_blueprint(capsule_bp, url_prefix='/api/capsule')
    
    # Database setup
    db.init_app(app)
    with app.app_context():
        db.create_all()
    
    # Serve frontend
    @app.route('/', defaults={'path': ''})
    @app.route('/<path:path>')
    def serve(path):
        static_folder_path = app.static_folder
        if static_folder_path is None:
            return "Static folder not configured", 404

        if path != "" and os.path.exists(os.path.join(static_folder_path, path)):
            return send_from_directory(static_folder_path, path)
        else:
            index_path = os.path.join(static_folder_path, 'index.html')
            if os.path.exists(index_path):
                return send_from_directory(static_folder_path, 'index.html')
            else:
                return "index.html not found", 404
    
    return app

app = create_app()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=False)
```

### 4. Create Gunicorn Configuration

Create `gunicorn.conf.py`:

```python
# Gunicorn configuration file
bind = "127.0.0.1:5000"
workers = 4
worker_class = "sync"
worker_connections = 1000
max_requests = 1000
max_requests_jitter = 100
timeout = 30
keepalive = 2
preload_app = True
user = "irys-app"
group = "irys-app"
tmp_upload_dir = None
secure_scheme_headers = {
    'X-FORWARDED-PROTOCOL': 'ssl',
    'X-FORWARDED-PROTO': 'https',
    'X-FORWARDED-SSL': 'on'
}
```

## Frontend Build and Integration

### 1. Build Frontend for Production

```bash
cd /home/irys-app/irys-time-capsule/frontend

# Install dependencies
npm install

# Build for production
npm run build
```

### 2. Copy Build to Backend

```bash
# Copy built files to backend static directory
cp -r dist/* ../backend/src/static/

# Verify files are copied
ls -la ../backend/src/static/
```

### 3. Configure Build Settings

Update `vite.config.js` for production:

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
  build: {
    outDir: 'dist',
    assetsDir: 'assets',
    sourcemap: false,
    minify: 'terser',
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          ethers: ['ethers'],
          ui: ['@radix-ui/react-dialog', '@radix-ui/react-select']
        }
      }
    }
  },
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:5000',
        changeOrigin: true
      }
    }
  }
})
```

## Production Server Setup

### 1. Create PM2 Ecosystem File

Create `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [{
    name: 'irys-time-capsule',
    cwd: '/home/irys-app/irys-time-capsule/backend',
    script: 'gunicorn',
    args: '-c gunicorn.conf.py src.main:app',
    interpreter: '/home/irys-app/irys-time-capsule/backend/venv/bin/python',
    instances: 1,
    exec_mode: 'fork',
    watch: false,
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'production',
      FLASK_ENV: 'production'
    },
    error_file: '/home/irys-app/logs/err.log',
    out_file: '/home/irys-app/logs/out.log',
    log_file: '/home/irys-app/logs/combined.log',
    time: true
  }]
};
```

### 2. Create Log Directory

```bash
mkdir -p /home/irys-app/logs
```

### 3. Start Application with PM2

```bash
# Start the application
pm2 start ecosystem.config.js

# Save PM2 configuration
pm2 save

# Setup PM2 startup script
pm2 startup
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u irys-app --hp /home/irys-app
```

### 4. Configure Nginx Reverse Proxy

Create `/etc/nginx/sites-available/irys-time-capsule`:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;
    
    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # Security Headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
    
    # Gzip Compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied expired no-cache no-store private must-revalidate auth;
    gzip_types text/plain text/css text/xml text/javascript application/x-javascript application/xml+rss application/javascript application/json;
    
    # Client Max Body Size
    client_max_body_size 50M;
    
    # Proxy to Flask application
    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port $server_port;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
    
    # Static file caching
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # API rate limiting
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Rate limiting zone
http {
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
}
```

### 5. Enable Nginx Site

```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/irys-time-capsule /etc/nginx/sites-enabled/

# Test Nginx configuration
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
```

## Domain and SSL Configuration

### 1. DNS Configuration

Configure your domain's DNS records:

```
Type: A
Name: @
Value: YOUR_SERVER_IP

Type: A
Name: www
Value: YOUR_SERVER_IP

Type: CNAME
Name: api
Value: yourdomain.com
```

### 2. SSL Certificate with Let's Encrypt

```bash
# Obtain SSL certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Test automatic renewal
sudo certbot renew --dry-run

# Setup automatic renewal
sudo crontab -e
# Add: 0 12 * * * /usr/bin/certbot renew --quiet
```

### 3. Firewall Configuration

```bash
# Configure UFW firewall
sudo ufw allow ssh
sudo ufw allow 'Nginx Full'
sudo ufw enable

# Check status
sudo ufw status
```

## Monitoring and Maintenance

### 1. Application Monitoring

Create monitoring script `monitor.sh`:

```bash
#!/bin/bash

# Check PM2 status
echo "=== PM2 Status ==="
pm2 status

# Check Nginx status
echo "=== Nginx Status ==="
sudo systemctl status nginx

# Check disk usage
echo "=== Disk Usage ==="
df -h

# Check memory usage
echo "=== Memory Usage ==="
free -h

# Check application logs
echo "=== Recent Application Logs ==="
pm2 logs irys-time-capsule --lines 10
```

### 2. Log Rotation

Configure log rotation in `/etc/logrotate.d/irys-time-capsule`:

```
/home/irys-app/logs/*.log {
    daily
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    create 644 irys-app irys-app
    postrotate
        pm2 reload irys-time-capsule
    endscript
}
```

### 3. Backup Strategy

Create backup script `backup.sh`:

```bash
#!/bin/bash

BACKUP_DIR="/home/irys-app/backups"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup database
cp /home/irys-app/irys-time-capsule/backend/src/database/app.db $BACKUP_DIR/app_$DATE.db

# Backup configuration
tar -czf $BACKUP_DIR/config_$DATE.tar.gz /etc/nginx/sites-available/irys-time-capsule /home/irys-app/irys-time-capsule/ecosystem.config.js

# Clean old backups (keep 30 days)
find $BACKUP_DIR -name "*.db" -mtime +30 -delete
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $DATE"
```

### 4. Health Check Endpoint

Add health check to your Flask application:

```python
@app.route('/health')
def health_check():
    return {
        'status': 'healthy',
        'timestamp': datetime.now().isoformat(),
        'version': '1.0.0'
    }
```

### 5. Automated Deployment Script

Create `deploy.sh` for updates:

```bash
#!/bin/bash

echo "Starting deployment..."

# Pull latest code
git pull origin main

# Update backend dependencies
cd backend
source venv/bin/activate
pip install -r requirements.txt

# Build frontend
cd ../frontend
npm install
npm run build

# Copy frontend build
cp -r dist/* ../backend/src/static/

# Restart application
pm2 restart irys-time-capsule

echo "Deployment completed!"
```

## Troubleshooting

### Common Issues

#### 1. PM2 Application Won't Start
```bash
# Check PM2 logs
pm2 logs irys-time-capsule

# Check Python virtual environment
source /home/irys-app/irys-time-capsule/backend/venv/bin/activate
python -c "import flask; print('Flask OK')"

# Restart application
pm2 restart irys-time-capsule
```

#### 2. Nginx 502 Bad Gateway
```bash
# Check if backend is running
curl http://127.0.0.1:5000/health

# Check Nginx error logs
sudo tail -f /var/log/nginx/error.log

# Test Nginx configuration
sudo nginx -t
```

#### 3. SSL Certificate Issues
```bash
# Check certificate status
sudo certbot certificates

# Renew certificate manually
sudo certbot renew

# Check Nginx SSL configuration
sudo nginx -t
```

#### 4. High Memory Usage
```bash
# Check memory usage
free -h
pm2 monit

# Restart application to clear memory
pm2 restart irys-time-capsule
```

### Performance Optimization

#### 1. Database Optimization
```bash
# For SQLite, enable WAL mode
sqlite3 app.db "PRAGMA journal_mode=WAL;"
```

#### 2. Nginx Caching
Add to Nginx configuration:
```nginx
# Enable caching
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=app_cache:10m max_size=1g inactive=60m use_temp_path=off;

location / {
    proxy_cache app_cache;
    proxy_cache_valid 200 1h;
    proxy_cache_use_stale error timeout invalid_header updating http_500 http_502 http_503 http_504;
}
```

#### 3. Application Optimization
```python
# Add Redis caching
from flask_caching import Cache

cache = Cache(app, config={'CACHE_TYPE': 'redis'})

@cache.memoize(timeout=300)
def get_cached_data():
    # Expensive operation
    pass
```

This deployment guide provides a comprehensive approach to deploying the Irys Time Capsule application in a production environment with proper security, monitoring, and maintenance procedures.

