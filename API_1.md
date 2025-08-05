# API Documentation - Irys Time Capsule

This document provides comprehensive documentation for the Irys Time Capsule REST API endpoints.

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Base URL](#base-url)
- [Response Format](#response-format)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)
- [Endpoints](#endpoints)
- [Examples](#examples)
- [SDK Integration](#sdk-integration)

## Overview

The Irys Time Capsule API provides RESTful endpoints for managing time capsules, including content upload to Irys storage, metadata management, and smart contract interaction. The API serves as a bridge between the frontend application and the Irys blockchain infrastructure.

### Key Features

- **Content Upload**: Upload text, images, and videos to Irys permanent storage
- **Metadata Management**: Create and retrieve metadata JSON for time capsules
- **Smart Contract Integration**: Provide contract ABI and deployment information
- **CORS Support**: Cross-origin requests enabled for frontend integration
- **Error Handling**: Comprehensive error responses with detailed messages

## Authentication

Currently, the API does not require authentication for basic operations. Smart contract interactions are authenticated through wallet signatures on the frontend. Future versions may implement API key authentication for enhanced security.

## Base URL

```
Production: https://yourdomain.com/api/capsule
Development: http://localhost:5000/api/capsule
```

## Response Format

All API responses follow a consistent JSON format:

### Success Response
```json
{
  "success": true,
  "data": {
    // Response data
  },
  "message": "Optional success message"
}
```

### Error Response
```json
{
  "success": false,
  "error": "Error description",
  "code": "ERROR_CODE",
  "details": {
    // Additional error details
  }
}
```

## Error Handling

### HTTP Status Codes

- **200 OK**: Request successful
- **400 Bad Request**: Invalid request parameters
- **404 Not Found**: Resource not found
- **429 Too Many Requests**: Rate limit exceeded
- **500 Internal Server Error**: Server error

### Error Codes

- **INVALID_CONTENT_TYPE**: Unsupported content type
- **FILE_TOO_LARGE**: File exceeds size limit
- **MISSING_REQUIRED_FIELD**: Required parameter missing
- **INVALID_DATE_FORMAT**: Date format incorrect
- **UPLOAD_FAILED**: Content upload to Irys failed
- **METADATA_ERROR**: Metadata creation failed

## Rate Limiting

API endpoints are rate-limited to prevent abuse:

- **Default Limit**: 100 requests per hour per IP
- **Upload Endpoints**: 10 requests per minute per IP
- **Headers**: Rate limit information included in response headers

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
```

## Endpoints

### Health Check

Check API status and connectivity.

```http
GET /health
```

**Response:**
```json
{
  "success": true,
  "message": "Irys Time Capsule API is running",
  "timestamp": "2025-07-28T15:20:30.439520",
  "version": "1.0.0"
}
```

---

### Upload Content

Upload content (text, image, or video) to Irys storage.

```http
POST /upload-content
```

**Content-Type:** `multipart/form-data`

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| content_type | string | Yes | Type of content: "text", "image", or "video" |
| content | string | Conditional | Text content (required if content_type is "text") |
| file | file | Conditional | File upload (required if content_type is "image" or "video") |

**Example Request (Text):**
```bash
curl -X POST http://localhost:5000/api/capsule/upload-content \
  -F "content_type=text" \
  -F "content=Hello from the past! This is my time capsule message."
```

**Example Request (File):**
```bash
curl -X POST http://localhost:5000/api/capsule/upload-content \
  -F "content_type=image" \
  -F "file=@/path/to/image.jpg"
```

**Response:**
```json
{
  "success": true,
  "content_uri": "irys://QmWXYZ789ABC123DEF456",
  "content_type": "text",
  "size": 1024,
  "upload_timestamp": "2025-07-28T15:20:30.439520"
}
```

**Error Responses:**
```json
{
  "success": false,
  "error": "No text content provided",
  "code": "MISSING_REQUIRED_FIELD"
}
```

---

### Create Metadata

Create and upload metadata JSON to Irys storage.

```http
POST /create-metadata
```

**Content-Type:** `application/json`

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| title | string | Yes | Human-readable title for the capsule |
| owner | string | Yes | Ethereum address of the capsule owner |
| unlock_date | string | Yes | ISO 8601 formatted unlock date |
| description | string | Yes | Detailed description of capsule contents |
| content_uri | string | Yes | Irys URI pointing to the content |

**Example Request:**
```bash
curl -X POST http://localhost:5000/api/capsule/create-metadata \
  -H "Content-Type: application/json" \
  -d '{
    "title": "My 2025 New Year Message",
    "owner": "0x742d35Cc6634C0532925a3b8D4C2C4e0C5e8c8e8",
    "unlock_date": "2028-01-01T00:00:00Z",
    "description": "A message to my future self with hopes and dreams.",
    "content_uri": "irys://QmWXYZ789ABC123DEF456"
  }'
```

**Response:**
```json
{
  "success": true,
  "metadata_uri": "irys://QmABC123DEF456GHI789",
  "metadata": {
    "title": "My 2025 New Year Message",
    "owner": "0x742d35Cc6634C0532925a3b8D4C2C4e0C5e8c8e8",
    "unlock_date": "2028-01-01T00:00:00Z",
    "description": "A message to my future self with hopes and dreams.",
    "content_uri": "irys://QmWXYZ789ABC123DEF456"
  },
  "upload_timestamp": "2025-07-28T15:20:30.439520"
}
```

**Error Responses:**
```json
{
  "success": false,
  "error": "Invalid unlock_date format. Use ISO 8601 format.",
  "code": "INVALID_DATE_FORMAT"
}
```

---

### Get Metadata

Retrieve metadata from Irys storage using the metadata URI.

```http
GET /get-metadata/{metadata_uri}
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| metadata_uri | string | Yes | URL-encoded Irys URI for the metadata |

**Example Request:**
```bash
curl -X GET "http://localhost:5000/api/capsule/get-metadata/irys%3A%2F%2FQmABC123DEF456GHI789"
```

**Response:**
```json
{
  "success": true,
  "metadata": {
    "title": "My 2025 New Year Message",
    "owner": "0x742d35Cc6634C0532925a3b8D4C2C4e0C5e8c8e8",
    "unlock_date": "2028-01-01T00:00:00Z",
    "description": "A message to my future self with hopes and dreams.",
    "content_uri": "irys://QmWXYZ789ABC123DEF456"
  },
  "retrieved_at": "2025-07-28T15:20:30.439520"
}
```

**Error Responses:**
```json
{
  "success": false,
  "error": "Metadata not found",
  "code": "NOT_FOUND"
}
```

---

### Get Content

Retrieve content from Irys storage using the content URI.

```http
GET /get-content/{content_uri}
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| content_uri | string | Yes | URL-encoded Irys URI for the content |

**Example Request:**
```bash
curl -X GET "http://localhost:5000/api/capsule/get-content/irys%3A%2F%2FQmWXYZ789ABC123DEF456"
```

**Response (Text Content):**
```json
{
  "success": true,
  "content": {
    "type": "text",
    "data": "Hello from the past! This is my time capsule message.",
    "encoding": "utf-8"
  },
  "retrieved_at": "2025-07-28T15:20:30.439520"
}
```

**Response (Binary Content):**
```json
{
  "success": true,
  "content": {
    "type": "image",
    "data": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQ...",
    "mime_type": "image/jpeg",
    "size": 245760
  },
  "retrieved_at": "2025-07-28T15:20:30.439520"
}
```

**Error Responses:**
```json
{
  "success": false,
  "error": "Content not found",
  "code": "NOT_FOUND"
}
```

---

### Contract Information

Get smart contract ABI and deployment information for frontend integration.

```http
GET /contract-info
```

**Example Request:**
```bash
curl -X GET http://localhost:5000/api/capsule/contract-info
```

**Response:**
```json
{
  "success": true,
  "contract_info": {
    "contract_address": "0x1234567890123456789012345678901234567890",
    "abi": [
      {
        "inputs": [
          {"internalType": "string", "name": "_title", "type": "string"},
          {"internalType": "uint256", "name": "_unlockTimestamp", "type": "uint256"},
          {"internalType": "string", "name": "_description", "type": "string"},
          {"internalType": "string", "name": "_contentURI", "type": "string"}
        ],
        "name": "createCapsule",
        "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}],
        "stateMutability": "payable",
        "type": "function"
      }
    ],
    "network": {
      "name": "Irys Testnet",
      "chain_id": 1337,
      "rpc_url": "https://rpc.irys.xyz",
      "explorer_url": "https://explorer.irys.xyz"
    },
    "deployment": {
      "block_number": 1234567,
      "transaction_hash": "0xabc123def456...",
      "deployed_at": "2025-07-28T10:00:00.000Z"
    }
  }
}
```

---

### Batch Operations

Upload multiple files or create multiple metadata entries in a single request.

```http
POST /batch-upload
```

**Content-Type:** `multipart/form-data`

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| operations | string | Yes | JSON array of upload operations |
| file_0, file_1, ... | file | Conditional | Files referenced in operations |

**Example Request:**
```bash
curl -X POST http://localhost:5000/api/capsule/batch-upload \
  -F "operations=[{\"type\":\"text\",\"content\":\"Message 1\"},{\"type\":\"image\",\"file_index\":0}]" \
  -F "file_0=@/path/to/image.jpg"
```

**Response:**
```json
{
  "success": true,
  "results": [
    {
      "index": 0,
      "success": true,
      "content_uri": "irys://QmABC123...",
      "content_type": "text"
    },
    {
      "index": 1,
      "success": true,
      "content_uri": "irys://QmDEF456...",
      "content_type": "image"
    }
  ],
  "total_uploaded": 2,
  "failed_uploads": 0
}
```

---

### Statistics

Get usage statistics and metrics.

```http
GET /stats
```

**Example Request:**
```bash
curl -X GET http://localhost:5000/api/capsule/stats
```

**Response:**
```json
{
  "success": true,
  "stats": {
    "total_uploads": 1250,
    "total_size_bytes": 52428800,
    "content_types": {
      "text": 800,
      "image": 350,
      "video": 100
    },
    "uploads_today": 45,
    "average_file_size": 41943,
    "uptime_seconds": 86400
  }
}
```

## Examples

### Complete Time Capsule Creation Flow

This example demonstrates the complete flow of creating a time capsule:

```javascript
// 1. Upload content
const uploadContent = async (content, type) => {
  const formData = new FormData();
  formData.append('content_type', type);
  
  if (type === 'text') {
    formData.append('content', content);
  } else {
    formData.append('file', content);
  }
  
  const response = await fetch('/api/capsule/upload-content', {
    method: 'POST',
    body: formData
  });
  
  return response.json();
};

// 2. Create metadata
const createMetadata = async (title, owner, unlockDate, description, contentUri) => {
  const response = await fetch('/api/capsule/create-metadata', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      title,
      owner,
      unlock_date: unlockDate,
      description,
      content_uri: contentUri
    })
  });
  
  return response.json();
};

// 3. Complete flow
const createTimeCapsule = async () => {
  try {
    // Upload content
    const contentResult = await uploadContent('Hello future me!', 'text');
    if (!contentResult.success) {
      throw new Error(contentResult.error);
    }
    
    // Create metadata
    const metadataResult = await createMetadata(
      'My Time Capsule',
      '0x742d35Cc6634C0532925a3b8D4C2C4e0C5e8c8e8',
      '2028-01-01T00:00:00Z',
      'A message to my future self',
      contentResult.content_uri
    );
    
    if (!metadataResult.success) {
      throw new Error(metadataResult.error);
    }
    
    // Now use metadataResult.metadata_uri with smart contract
    console.log('Metadata URI:', metadataResult.metadata_uri);
    
  } catch (error) {
    console.error('Error creating time capsule:', error);
  }
};
```

### Error Handling Example

```javascript
const handleApiCall = async (url, options) => {
  try {
    const response = await fetch(url, options);
    const data = await response.json();
    
    if (!data.success) {
      // Handle API-level errors
      switch (data.code) {
        case 'INVALID_CONTENT_TYPE':
          throw new Error('Please select a valid file type');
        case 'FILE_TOO_LARGE':
          throw new Error('File size exceeds 50MB limit');
        case 'MISSING_REQUIRED_FIELD':
          throw new Error('Please fill in all required fields');
        default:
          throw new Error(data.error || 'Unknown error occurred');
      }
    }
    
    return data;
    
  } catch (error) {
    if (error.name === 'TypeError') {
      // Network error
      throw new Error('Network error. Please check your connection.');
    }
    throw error;
  }
};
```

### Rate Limiting Handling

```javascript
const apiCallWithRetry = async (url, options, maxRetries = 3) => {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);
      
      if (response.status === 429) {
        // Rate limited
        const retryAfter = response.headers.get('Retry-After') || 60;
        console.log(`Rate limited. Retrying after ${retryAfter} seconds...`);
        await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
        continue;
      }
      
      return response.json();
      
    } catch (error) {
      if (attempt === maxRetries) {
        throw error;
      }
      console.log(`Attempt ${attempt} failed. Retrying...`);
      await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
    }
  }
};
```

## SDK Integration

### JavaScript/TypeScript SDK

```typescript
interface TimeCapsuleAPI {
  uploadContent(content: string | File, type: 'text' | 'image' | 'video'): Promise<UploadResult>;
  createMetadata(metadata: MetadataInput): Promise<MetadataResult>;
  getMetadata(uri: string): Promise<Metadata>;
  getContent(uri: string): Promise<Content>;
  getContractInfo(): Promise<ContractInfo>;
}

class TimeCapsuleClient implements TimeCapsuleAPI {
  constructor(private baseUrl: string) {}
  
  async uploadContent(content: string | File, type: 'text' | 'image' | 'video'): Promise<UploadResult> {
    const formData = new FormData();
    formData.append('content_type', type);
    
    if (typeof content === 'string') {
      formData.append('content', content);
    } else {
      formData.append('file', content);
    }
    
    const response = await fetch(`${this.baseUrl}/upload-content`, {
      method: 'POST',
      body: formData
    });
    
    const result = await response.json();
    if (!result.success) {
      throw new Error(result.error);
    }
    
    return result;
  }
  
  // ... other methods
}

// Usage
const client = new TimeCapsuleClient('https://yourdomain.com/api/capsule');
const result = await client.uploadContent('Hello world!', 'text');
```

### Python SDK

```python
import requests
from typing import Union, Dict, Any

class TimeCapsuleClient:
    def __init__(self, base_url: str):
        self.base_url = base_url.rstrip('/')
    
    def upload_content(self, content: Union[str, bytes], content_type: str) -> Dict[str, Any]:
        """Upload content to Irys storage."""
        url = f"{self.base_url}/upload-content"
        
        if content_type == 'text':
            data = {'content_type': content_type, 'content': content}
            response = requests.post(url, data=data)
        else:
            files = {'file': content}
            data = {'content_type': content_type}
            response = requests.post(url, data=data, files=files)
        
        result = response.json()
        if not result.get('success'):
            raise Exception(result.get('error', 'Upload failed'))
        
        return result
    
    def create_metadata(self, title: str, owner: str, unlock_date: str, 
                       description: str, content_uri: str) -> Dict[str, Any]:
        """Create and upload metadata."""
        url = f"{self.base_url}/create-metadata"
        
        payload = {
            'title': title,
            'owner': owner,
            'unlock_date': unlock_date,
            'description': description,
            'content_uri': content_uri
        }
        
        response = requests.post(url, json=payload)
        result = response.json()
        
        if not result.get('success'):
            raise Exception(result.get('error', 'Metadata creation failed'))
        
        return result

# Usage
client = TimeCapsuleClient('https://yourdomain.com/api/capsule')
result = client.upload_content('Hello world!', 'text')
print(f"Content URI: {result['content_uri']}")
```

This API documentation provides comprehensive information for integrating with the Irys Time Capsule backend services, including detailed examples and error handling patterns for robust application development.

