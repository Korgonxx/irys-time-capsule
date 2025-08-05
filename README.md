# Irys Time Capsule

A decentralized application (dApp) built on Irys, a programmable datachain with EVM compatibility and low-cost permanent storage. The application allows users to create and retrieve digital time capsules that unlock after a set date, storing text, images, or small video files permanently on the Irys network.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [Installation](#installation)
- [Usage](#usage)
- [Smart Contract](#smart-contract)
- [API Documentation](#api-documentation)
- [Deployment](#deployment)
- [Development](#development)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Overview

Irys Time Capsule enables users to preserve digital memories for the future through blockchain-enforced time locks. Users can upload content to the permanent Irys datachain, set an unlock date, and retrieve their capsules only after the specified time has passed. This creates a unique way to send messages, photos, or videos to your future self or others.

### Key Concepts

**Time-Locked Storage**: Content remains inaccessible until a predetermined unlock date, enforced by smart contract logic.

**Permanent Storage**: All content is stored on the Irys datachain, ensuring it remains accessible indefinitely without risk of data loss.

**Decentralized Access Control**: Smart contracts manage access permissions, eliminating the need for centralized authorities.

**Programmable Metadata**: Unlock conditions are embedded in metadata using programmable tags, allowing for complex retrieval logic.

## Features

### Core Functionality

- **Create Time Capsules**: Upload text messages, images, or videos with custom unlock dates
- **Time-Lock Enforcement**: Smart contracts prevent access until unlock conditions are met
- **Permanent Storage**: Content stored on Irys datachain with guaranteed permanence
- **Wallet Integration**: MetaMask support for seamless blockchain interaction
- **Real-time Countdown**: Visual timers showing time remaining until unlock
- **Content Retrieval**: Access unlocked capsules and view their contents
- **Fee Management**: Configurable creation fees to prevent spam

### User Interface

- **Futuristic Dark Theme**: Modern, high-contrast design optimized for the blockchain aesthetic
- **Responsive Design**: Works seamlessly on desktop and mobile devices
- **Interactive Elements**: Smooth animations and micro-interactions for enhanced user experience
- **Status Indicators**: Clear visual feedback for capsule states (locked/unlocked)
- **Progress Tracking**: Real-time updates on capsule creation and retrieval processes

### Technical Features

- **EVM Compatibility**: Built for Irys blockchain with Ethereum-compatible smart contracts
- **Cross-Origin Support**: CORS-enabled API for frontend-backend communication
- **Error Handling**: Comprehensive error management and user feedback
- **Gas Optimization**: Efficient smart contract design to minimize transaction costs
- **Scalable Architecture**: Modular design supporting future feature additions

## Architecture

The Irys Time Capsule follows a three-tier architecture comprising the frontend interface, backend API services, and blockchain smart contracts, with permanent storage provided by the Irys datachain.

### System Components

**Frontend (React)**: User interface built with React, providing wallet connection, capsule creation forms, and content viewing capabilities. The frontend communicates with both the backend API and blockchain smart contracts.

**Backend (Flask)**: RESTful API server handling content uploads to Irys, metadata management, and serving as a bridge between the frontend and blockchain. The backend also serves the built frontend application for production deployment.

**Smart Contracts (Solidity)**: Ethereum-compatible contracts deployed on Irys blockchain, managing capsule creation, time-lock enforcement, and retrieval permissions.

**Storage Layer (Irys)**: Permanent storage for both content and metadata, with programmable tags enabling complex unlock conditions.

### Data Flow

1. **Content Upload**: User uploads content through frontend, which sends it to backend API
2. **Irys Storage**: Backend uploads content to Irys and receives permanent URI
3. **Metadata Creation**: Backend generates metadata JSON with unlock conditions and uploads to Irys
4. **Smart Contract Interaction**: Frontend calls smart contract to create capsule record with metadata URI
5. **Time-Lock Enforcement**: Smart contract prevents retrieval until unlock timestamp is reached
6. **Content Retrieval**: After unlock time, users can retrieve capsules and access content via Irys URIs

## Technology Stack

### Blockchain & Smart Contracts
- **Solidity**: Smart contract development language
- **Hardhat**: Development environment and testing framework
- **Ethers.js**: Ethereum library for blockchain interaction
- **MetaMask**: Wallet integration for user authentication

### Backend Services
- **Flask**: Python web framework for API development
- **Flask-CORS**: Cross-origin resource sharing support
- **Web3.py**: Python library for blockchain interaction
- **Python 3.11**: Runtime environment

### Frontend Application
- **React**: User interface framework
- **Vite**: Build tool and development server
- **Tailwind CSS**: Utility-first CSS framework
- **shadcn/ui**: Component library for consistent design
- **Lucide Icons**: Icon library for modern interface elements

### Storage & Infrastructure
- **Irys**: Permanent data storage and programmable datachain
- **Node.js**: JavaScript runtime for build tools
- **pnpm**: Package manager for efficient dependency management

## Installation

### Prerequisites

Before installing Irys Time Capsule, ensure you have the following software installed on your system:

- **Node.js** (version 18 or higher)
- **Python** (version 3.11 or higher)
- **Git** for version control
- **MetaMask** browser extension for wallet functionality

### Clone Repository

```bash
git clone https://github.com/your-username/irys-time-capsule.git
cd irys-time-capsule
```

### Backend Setup

Navigate to the backend directory and set up the Python environment:

```bash
cd backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### Frontend Setup

Navigate to the frontend directory and install dependencies:

```bash
cd frontend
pnpm install
```

### Smart Contract Setup

Navigate to the project root and install Hardhat dependencies:

```bash
npm install
```

## Usage

### Starting the Development Environment

1. **Start Backend Server**:
```bash
cd backend
source venv/bin/activate
python src/main.py
```

2. **Start Frontend Development Server** (in a new terminal):
```bash
cd frontend
pnpm run dev
```

3. **Access Application**:
Open your browser and navigate to `http://localhost:5173`

### Creating a Time Capsule

1. **Connect Wallet**: Click "Connect MetaMask" and approve the connection
2. **Navigate to Create Tab**: Click on "Create New" in the application
3. **Fill Capsule Details**:
   - Enter a descriptive title
   - Write a detailed description
   - Set the unlock date (must be in the future)
   - Choose content type (text, image, or video)
   - Add your content or upload a file
4. **Submit Transaction**: Click "Create Time Capsule" and approve the transaction in MetaMask
5. **Confirmation**: Wait for transaction confirmation and Irys upload completion

### Viewing Your Capsules

1. **Navigate to Capsules Tab**: Click on "My Capsules" to view your created capsules
2. **Check Status**: View countdown timers for locked capsules or "Unlocked" status
3. **Retrieve Capsules**: For unlocked capsules, click "Retrieve" to mark them as accessed
4. **View Content**: After retrieval, click "View Content" to access the stored data

### Managing Capsules

**Locked Capsules**: Display countdown timers and cannot be accessed until unlock time
**Unlocked Capsules**: Show "Retrieve" button to mark as accessed
**Retrieved Capsules**: Display "View Content" button for accessing stored data

## Smart Contract

The TimeCapsule smart contract manages the core functionality of time-locked storage on the Irys blockchain.

### Contract Functions

#### createCapsule
```solidity
function createCapsule(
    string memory _title,
    uint256 _unlockTimestamp,
    string memory _description,
    string memory _contentURI
) public payable returns (uint256)
```

Creates a new time capsule with the specified parameters. Requires payment of the creation fee and validates that the unlock timestamp is in the future.

**Parameters**:
- `_title`: Human-readable title for the capsule
- `_unlockTimestamp`: Unix timestamp when capsule can be retrieved
- `_description`: Detailed description of capsule contents
- `_contentURI`: Irys URI pointing to the metadata JSON

**Returns**: Unique capsule ID for future reference

#### retrieveCapsule
```solidity
function retrieveCapsule(uint256 _capsuleId) public
```

Marks a capsule as retrieved, allowing access to its contents. Can only be called by the capsule owner after the unlock timestamp has passed.

**Parameters**:
- `_capsuleId`: ID of the capsule to retrieve

#### capsuleLocked
```solidity
function capsuleLocked(uint256 _capsuleId) public view returns (bool)
```

Checks if a capsule is still locked based on current time and retrieval status.

**Parameters**:
- `_capsuleId`: ID of the capsule to check

**Returns**: Boolean indicating if capsule is locked

#### getCapsule
```solidity
function getCapsule(uint256 _capsuleId) public view returns (
    address owner,
    string memory title,
    uint256 unlockTimestamp,
    string memory description,
    string memory contentURI,
    bool isLocked
)
```

Retrieves complete information about a specific capsule.

**Parameters**:
- `_capsuleId`: ID of the capsule to query

**Returns**: Tuple containing all capsule data

### Events

#### CapsuleCreated
```solidity
event CapsuleCreated(uint256 indexed capsuleId, address indexed owner, string title, uint256 unlockTimestamp)
```

Emitted when a new capsule is created.

#### CapsuleRetrieved
```solidity
event CapsuleRetrieved(uint256 indexed capsuleId, address indexed owner)
```

Emitted when a capsule is successfully retrieved.

### Deployment

To deploy the smart contract to Irys testnet:

```bash
npx hardhat compile
npx hardhat test
npx hardhat run scripts/deploy.js --network irys-testnet
```

## API Documentation

The backend provides RESTful API endpoints for content management and blockchain interaction.

### Base URL
```
http://localhost:5000/api/capsule
```

### Endpoints

#### Health Check
```http
GET /health
```

Returns API status and timestamp.

**Response**:
```json
{
  "success": true,
  "message": "Irys Time Capsule API is running",
  "timestamp": "2025-07-28T15:20:30.439520"
}
```

#### Upload Content
```http
POST /upload-content
```

Uploads content to Irys storage.

**Request Body** (multipart/form-data):
- `content_type`: "text", "image", or "video"
- `content`: Text content (for text type)
- `file`: File upload (for image/video types)

**Response**:
```json
{
  "success": true,
  "content_uri": "irys://content_hash",
  "content_type": "text"
}
```

#### Create Metadata
```http
POST /create-metadata
```

Creates and uploads metadata JSON to Irys.

**Request Body** (JSON):
```json
{
  "title": "My Time Capsule",
  "owner": "0x742d35Cc6634C0532925a3b8D4C2C4e0C5e8c8e8",
  "unlock_date": "2028-01-01T00:00:00Z",
  "description": "A message to my future self",
  "content_uri": "irys://content_hash"
}
```

**Response**:
```json
{
  "success": true,
  "metadata_uri": "irys://metadata_hash",
  "metadata": { /* metadata object */ }
}
```

#### Get Metadata
```http
GET /get-metadata/{metadata_uri}
```

Retrieves metadata from Irys storage.

**Response**:
```json
{
  "success": true,
  "metadata": {
    "title": "My Time Capsule",
    "owner": "0x742d35Cc6634C0532925a3b8D4C2C4e0C5e8c8e8",
    "unlock_date": "2028-01-01T00:00:00Z",
    "description": "A message to my future self",
    "content_uri": "irys://content_hash"
  }
}
```

#### Get Content
```http
GET /get-content/{content_uri}
```

Retrieves content from Irys storage.

**Response**:
```json
{
  "success": true,
  "content": {
    "type": "text",
    "data": "This is my time capsule message!"
  }
}
```

#### Contract Information
```http
GET /contract-info
```

Returns smart contract ABI and deployment information.

**Response**:
```json
{
  "success": true,
  "contract_info": {
    "contract_address": "0x1234567890123456789012345678901234567890",
    "abi": [ /* contract ABI */ ],
    "network": {
      "name": "Irys Testnet",
      "chain_id": 1337,
      "rpc_url": "https://rpc.irys.xyz"
    }
  }
}
```

## Deployment

### Production Build

1. **Build Frontend**:
```bash
cd frontend
pnpm run build
```

2. **Copy to Backend**:
```bash
cp -r frontend/dist/* backend/src/static/
```

3. **Update Dependencies**:
```bash
cd backend
source venv/bin/activate
pip freeze > requirements.txt
```

### Environment Configuration

Create environment variables for production:

```bash
export FLASK_ENV=production
export IRYS_PRIVATE_KEY=your_private_key
export CONTRACT_ADDRESS=deployed_contract_address
export IRYS_RPC_URL=https://rpc.irys.xyz
```

### Server Deployment

For production deployment, use a WSGI server like Gunicorn:

```bash
pip install gunicorn
gunicorn -w 4 -b 0.0.0.0:5000 src.main:app
```

### Smart Contract Deployment

1. **Configure Network**:
Update `hardhat.config.js` with Irys network configuration:

```javascript
module.exports = {
  networks: {
    irys: {
      url: "https://rpc.irys.xyz",
      accounts: [process.env.PRIVATE_KEY]
    }
  }
};
```

2. **Deploy Contract**:
```bash
npx hardhat run scripts/deploy.js --network irys
```

3. **Verify Contract**:
```bash
npx hardhat verify --network irys DEPLOYED_CONTRACT_ADDRESS
```

## Development

### Project Structure

```
irys-time-capsule/
├── contracts/                 # Smart contracts
│   └── TimeCapsule.sol
├── test/                     # Contract tests
│   └── TimeCapsule.js
├── backend/                  # Flask API server
│   ├── src/
│   │   ├── routes/          # API endpoints
│   │   ├── models/          # Database models
│   │   ├── static/          # Frontend build files
│   │   └── main.py          # Application entry point
│   ├── venv/                # Python virtual environment
│   └── requirements.txt     # Python dependencies
├── frontend/                # React application
│   ├── src/
│   │   ├── components/      # React components
│   │   ├── hooks/           # Custom React hooks
│   │   ├── assets/          # Static assets
│   │   └── App.jsx          # Main application component
│   ├── dist/                # Build output
│   └── package.json         # Node.js dependencies
├── hardhat.config.js        # Hardhat configuration
├── package.json             # Project dependencies
└── README.md                # Documentation
```

### Testing

#### Smart Contract Tests
```bash
npx hardhat test
```

#### API Tests
```bash
cd backend
source venv/bin/activate
python -m pytest tests/
```

#### Frontend Tests
```bash
cd frontend
pnpm test
```

### Code Style

The project follows established coding standards:

- **Solidity**: Follows OpenZeppelin style guide
- **Python**: PEP 8 compliance with Black formatting
- **JavaScript/React**: ESLint configuration with Prettier
- **CSS**: Tailwind CSS utility classes

### Contributing Guidelines

1. **Fork Repository**: Create a personal fork of the project
2. **Create Branch**: Use descriptive branch names (feature/add-nft-support)
3. **Write Tests**: Include tests for new functionality
4. **Follow Style**: Adhere to project coding standards
5. **Submit PR**: Create pull request with detailed description

## Troubleshooting

### Common Issues

#### MetaMask Connection Failed
**Problem**: Wallet connection fails or shows errors
**Solution**: 
- Ensure MetaMask is installed and unlocked
- Check that you're on the correct network (Irys testnet)
- Clear browser cache and reload the page
- Try disconnecting and reconnecting the wallet

#### Transaction Fails
**Problem**: Smart contract transactions fail or revert
**Solution**:
- Verify you have sufficient balance for gas fees
- Check that unlock date is set in the future
- Ensure creation fee is included in transaction
- Verify contract address is correct

#### Content Upload Errors
**Problem**: File uploads fail or timeout
**Solution**:
- Check file size limits (recommended < 10MB)
- Verify internet connection stability
- Try uploading smaller files first
- Check backend server logs for detailed errors

#### Frontend Build Issues
**Problem**: Build process fails or produces errors
**Solution**:
- Clear node_modules and reinstall dependencies
- Check Node.js version compatibility
- Verify all environment variables are set
- Review build logs for specific error messages

#### Backend API Errors
**Problem**: API endpoints return 500 errors
**Solution**:
- Check Python virtual environment is activated
- Verify all dependencies are installed
- Review Flask application logs
- Ensure CORS is properly configured

### Debug Mode

Enable debug mode for detailed error information:

**Backend**:
```bash
export FLASK_DEBUG=1
python src/main.py
```

**Frontend**:
```bash
pnpm run dev
```

### Log Analysis

Check application logs for troubleshooting:

**Backend Logs**: Available in terminal output when running Flask
**Frontend Logs**: Available in browser developer console
**Smart Contract Logs**: Available through Hardhat network or block explorer

### Performance Optimization

For optimal performance:

- **Frontend**: Enable production build optimizations
- **Backend**: Use production WSGI server (Gunicorn)
- **Smart Contracts**: Optimize gas usage through efficient coding
- **Storage**: Compress large files before upload

## Contributing

We welcome contributions to the Irys Time Capsule project. Please read our contributing guidelines and submit pull requests for any improvements.

### Development Setup

1. Fork the repository
2. Clone your fork locally
3. Follow installation instructions
4. Create a feature branch
5. Make your changes
6. Add tests for new functionality
7. Submit a pull request

### Reporting Issues

Please use GitHub Issues to report bugs or request features. Include:
- Detailed description of the issue
- Steps to reproduce
- Expected vs actual behavior
- Environment information (OS, browser, versions)

## License

This project is licensed under the MIT License. See the LICENSE file for details.

---

**Built with ❤️ for the decentralized future**

For questions or support, please open an issue on GitHub or contact the development team.

