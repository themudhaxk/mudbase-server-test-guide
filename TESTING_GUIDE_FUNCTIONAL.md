# MUDBASE Functional Testing Guide

> **Purpose**: Comprehensive testing guide for all MUDBASE backend functionalities  
> **Audience**: QA Engineers, Developers, Product Managers  
> **Estimated Time**: 4-6 hours for complete testing

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Testing Environment Setup](#testing-environment-setup)
3. [Authentication & Authorization Testing](#authentication--authorization-testing)
4. [Multi-Role Feature Testing](#multi-role-feature-testing)
5. [Database Operations Testing](#database-operations-testing)
6. [File Storage Testing](#file-storage-testing)
7. [Real-time Features Testing](#real-time-features-testing)
8. [Billing & Payment Testing](#billing--payment-testing)
9. [Wallet Service Testing](#wallet-service-testing)
10. [Integration & Webhooks Testing](#integration--webhooks-testing)
11. [Chat System Testing](#chat-system-testing)
12. [Search Functionality Testing](#search-functionality-testing)
13. [Multi-Vendor Application Scenario](#multi-vendor-application-scenario)
14. [Test Checklist](#test-checklist)

---

## Prerequisites

### Required Tools

- **API Client**: Postman, Insomnia, or cURL
- **WebSocket Client**: Postman (WebSocket), wscat, or browser console
- **Browser**: Chrome/Firefox for webhooks testing
- **MongoDB Compass** (optional): For database verification
- **ngrok** (optional): For webhook testing

### Base URL

```
Development: http://localhost:5000
Staging: https://staging-mudbase-server.onrender.com
Production: https://mudbase-server.onrender.com
```

### Test Accounts Setup

You'll need to create test accounts for different roles:
- Organization Owner
- Admin
- Developer
- Viewer
- Customer (multi-role)
- Vendor (multi-role)
- Rider (multi-role)

---

## Testing Environment Setup

### Step 1: Create Test Organization

```bash
POST {{baseUrl}}/api/orgs
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "Test Organization",
  "slug": "test-org",
  "description": "Organization for functional testing"
}

# Expected Response: 201 Created
# Save orgId from response
```

**Test Case 1.1**: Organization Creation
- ✅ Verify organization is created with correct name
- ✅ Verify slug is generated correctly
- ✅ Verify creator becomes organization owner
- ✅ Verify organization appears in user's org list

### Step 2: Create Test Project

```bash
POST {{baseUrl}}/api/orgs/{{orgId}}/projects
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "Test Project",
  "description": "Project for functional testing",
  "settings": {
    "frontendUrl": "http://localhost:3000"
  },
  "auth": {
    "methods": ["local", "oauth"],
    "providers": []
  }
}

# Expected Response: 201 Created
# Save projectId from response
```

**Test Case 2.1**: Project Creation
- ✅ Verify project is created successfully
- ✅ Verify project belongs to correct organization
- ✅ Verify API keys are generated
- ✅ Verify project appears in projects list

### Step 3: Create Test Collections

```bash
# Collection 1: Users
POST {{baseUrl}}/api/projects/{{projectId}}/collections
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "users",
  "slug": "users",
  "fields": [
    {
      "name": "email",
      "type": "string",
      "required": true,
      "unique": true
    },
    {
      "name": "name",
      "type": "string",
      "required": true
    },
    {
      "name": "role",
      "type": "string",
      "required": false
    }
  ],
  "permissions": [
    {
      "role": "developer",
      "actions": ["create", "read", "update", "delete"]
    }
  ]
}

# Collection 2: Products
POST {{baseUrl}}/api/projects/{{projectId}}/collections
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "products",
  "slug": "products",
  "fields": [
    {
      "name": "name",
      "type": "string",
      "required": true
    },
    {
      "name": "price",
      "type": "number",
      "required": true
    },
    {
      "name": "status",
      "type": "string",
      "required": false,
      "default": "active"
    }
  ],
  "permissions": [
    {
      "role": "developer",
      "actions": ["create", "read", "update", "delete"]
    }
  ]
}

# Expected Response: 201 Created for each
# Save collectionIds from responses
```

**Test Case 3.1**: Collection Creation
- ✅ Verify collections are created successfully
- ✅ Verify fields are defined correctly
- ✅ Verify permissions are set correctly
- ✅ Verify collections appear in collections list

---

## Authentication & Authorization Testing

### Test Case 4.1: Local Registration

```bash
POST {{baseUrl}}/api/auth/local/register
Content-Type: application/json

{
  "email": "testuser@example.com",
  "password": "TestPassword123!",
  "firstName": "Test",
  "lastName": "User",
  "projectId": "{{projectId}}"
}

# Expected Response: 201 Created
# Save userId and token from response
```

**Validation**:
- ✅ User is created successfully
- ✅ Email verification token is generated
- ✅ Default role is assigned (developer)
- ✅ JWT token is returned
- ✅ Refresh token is returned

### Test Case 4.2: Email Verification

```bash
POST {{baseUrl}}/api/auth/verify-email
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "token": "{{verificationToken}}"
}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Email is marked as verified
- ✅ User can login after verification

### Test Case 4.3: Login

```bash
POST {{baseUrl}}/api/auth/local/login
Content-Type: application/json

{
  "email": "testuser@example.com",
  "password": "TestPassword123!",
  "projectId": "{{projectId}}"
}

# Expected Response: 200 OK
# Save new token from response
```

**Validation**:
- ✅ Valid credentials return JWT token
- ✅ Invalid credentials return 401
- ✅ Token contains user information
- ✅ Token expires correctly

### Test Case 4.4: Password Reset

```bash
# Step 1: Request Password Reset
POST {{baseUrl}}/api/auth/local/password-reset
Content-Type: application/json

{
  "email": "testuser@example.com",
  "projectId": "{{projectId}}"
}

# Expected Response: 200 OK

# Step 2: Reset Password (use token from email)
POST {{baseUrl}}/api/auth/password-reset/{{resetToken}}
Content-Type: application/json

{
  "password": "NewPassword123!"
}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Reset email is sent
- ✅ Password can be reset with valid token
- ✅ Old password no longer works
- ✅ New password works for login

### Test Case 4.5: Token Refresh

```bash
POST {{baseUrl}}/api/auth/refresh
Content-Type: application/json

{
  "refreshToken": "{{refreshToken}}"
}

# Expected Response: 200 OK
# New access token returned
```

**Validation**:
- ✅ New access token is returned
- ✅ New refresh token is returned
- ✅ Old refresh token is invalidated

---

## Multi-Role Feature Testing

### Test Case 5.1: Initialize Multi-Role Feature

```bash
GET {{baseUrl}}/api/projects/{{projectId}}/multi-role
Authorization: Bearer {{token}}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Feature auto-initializes with default roles
- ✅ Default roles are present (owner, admin, support, seller, vendor, rider, customer)
- ✅ All roles are enabled by default
- ✅ Default role is set to "customer"

### Test Case 5.2: Get Available Roles

```bash
GET {{baseUrl}}/api/projects/{{projectId}}/multi-role/roles/available
Authorization: Bearer {{token}}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Returns list of enabled roles
- ✅ Each role has slug, name, description, signupEndpoint
- ✅ Roles show requirements (approval, payment, KYC)

### Test Case 5.3: Role-Based Customer Signup

```bash
POST {{baseUrl}}/api/auth/local/signup/customer
Content-Type: application/json

{
  "email": "customer@example.com",
  "password": "CustomerPass123!",
  "firstName": "Customer",
  "lastName": "User",
  "projectId": "{{projectId}}"
}

# Expected Response: 201 Created
```

**Validation**:
- ✅ User is created successfully
- ✅ customRole is set to "customer"
- ✅ Role information is returned in response
- ✅ User can login with customer role

### Test Case 5.4: Role-Based Vendor Signup

```bash
POST {{baseUrl}}/api/auth/local/signup/vendor
Content-Type: application/json

{
  "email": "vendor@example.com",
  "password": "VendorPass123!",
  "firstName": "Vendor",
  "lastName": "User",
  "projectId": "{{projectId}}"
}

# Expected Response: 201 Created
```

**Validation**:
- ✅ User is created with vendor role
- ✅ customRole is set to "vendor"
- ✅ Role requirements are returned (payment, KYC)

### Test Case 5.5: Toggle Role On/Off

```bash
# Disable vendor role
PATCH {{baseUrl}}/api/projects/{{projectId}}/multi-role/roles/vendor/toggle
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "isEnabled": false
}

# Expected Response: 200 OK

# Try to signup with vendor role (should fail)
POST {{baseUrl}}/api/auth/local/signup/vendor
Content-Type: application/json

{
  "email": "vendor2@example.com",
  "password": "VendorPass123!",
  "firstName": "Vendor2",
  "lastName": "User",
  "projectId": "{{projectId}}"
}

# Expected Response: 404 Not Found (role not available)

# Re-enable vendor role
PATCH {{baseUrl}}/api/projects/{{projectId}}/multi-role/roles/vendor/toggle
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "isEnabled": true
}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Role can be disabled
- ✅ Disabled role cannot be used for signup
- ✅ Role can be re-enabled
- ✅ Enabled role works for signup

### Test Case 5.6: Update Role Configuration

```bash
PATCH {{baseUrl}}/api/projects/{{projectId}}/multi-role/roles/vendor
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "Vendor (Updated)",
  "description": "Updated vendor description",
  "requiresApproval": true,
  "requiresPayment": false,
  "requiresKYC": true
}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Role name is updated
- ✅ Description is updated
- ✅ Requirements are updated
- ✅ Changes are persisted

### Test Case 5.7: Add Custom Role

```bash
POST {{baseUrl}}/api/projects/{{projectId}}/multi-role/roles
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "slug": "premium_customer",
  "name": "Premium Customer",
  "description": "Premium customer with enhanced features",
  "signupEndpoint": "premium-customer",
  "hierarchy": 25,
  "defaultPermissions": [
    {
      "resource": "data",
      "actions": ["create", "read", "update"]
    }
  ],
  "requiresApproval": false,
  "requiresPayment": true
}

# Expected Response: 201 Created
```

**Validation**:
- ✅ Custom role is created
- ✅ Role appears in available roles list
- ✅ Role can be used for signup
- ✅ Permissions are set correctly

### Test Case 5.8: Configure Collection Permissions for Role

```bash
PATCH {{baseUrl}}/api/projects/{{projectId}}/multi-role/roles/vendor/collections/{{productsCollectionId}}/permissions
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "actions": ["create", "read", "update", "delete"],
  "conditions": {
    "vendorId": "$user.metadata.vendorId"
  }
}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Permissions are updated
- ✅ Conditions are set correctly
- ✅ Permissions are applied to collection
- ✅ Role users have correct access

---

## Database Operations Testing

### Test Case 6.1: Create Data

```bash
POST {{baseUrl}}/api/projects/{{projectId}}/collections/{{usersCollectionId}}/data
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "email": "newuser@example.com",
  "name": "New User",
  "role": "customer"
}

# Expected Response: 201 Created
# Save documentId from response
```

**Validation**:
- ✅ Document is created successfully
- ✅ All required fields are validated
- ✅ Unique constraints are enforced
- ✅ Timestamps are set (createdAt, updatedAt)

### Test Case 6.2: Read Data (Single)

```bash
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{usersCollectionId}}/data/{{documentId}}
Authorization: Bearer {{token}}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Document is returned correctly
- ✅ All fields are present
- ✅ Data matches created data

### Test Case 6.3: Read Data (List with Filters)

```bash
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{usersCollectionId}}/data?filter={"role":"customer"}&limit=10&skip=0&sort=-createdAt
Authorization: Bearer {{token}}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Filtered results are returned
- ✅ Pagination works (limit, skip)
- ✅ Sorting works correctly
- ✅ Total count is accurate

### Test Case 6.4: Update Data

```bash
PATCH {{baseUrl}}/api/projects/{{projectId}}/collections/{{usersCollectionId}}/data/{{documentId}}
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "Updated User Name",
  "role": "premium"
}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Document is updated successfully
- ✅ Only provided fields are updated
- ✅ updatedAt timestamp is changed
- ✅ Other fields remain unchanged

### Test Case 6.5: Delete Data

```bash
DELETE {{baseUrl}}/api/projects/{{projectId}}/collections/{{usersCollectionId}}/data/{{documentId}}
Authorization: Bearer {{token}}

# Expected Response: 200 OK

# Verify deletion
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{usersCollectionId}}/data/{{documentId}}
Authorization: Bearer {{token}}

# Expected Response: 404 Not Found
```

**Validation**:
- ✅ Document is deleted successfully
- ✅ Deleted document cannot be retrieved
- ✅ Related data integrity is maintained

### Test Case 6.6: Bulk Operations

```bash
# Bulk Create
POST {{baseUrl}}/api/projects/{{projectId}}/collections/{{usersCollectionId}}/data/bulk
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "operations": [
    {
      "email": "user1@example.com",
      "name": "User 1",
      "role": "customer"
    },
    {
      "email": "user2@example.com",
      "name": "User 2",
      "role": "customer"
    }
  ]
}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Multiple documents are created
- ✅ Partial failures are handled correctly
- ✅ Transaction integrity is maintained

---

## File Storage Testing

### Test Case 7.1: Create Bucket

```bash
POST {{baseUrl}}/api/projects/{{projectId}}/buckets
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "test-bucket",
  "isPublic": false,
  "allowedMimeTypes": ["image/jpeg", "image/png", "application/pdf"]
}

# Expected Response: 201 Created
# Save bucketId from response
```

**Validation**:
- ✅ Bucket is created successfully
- ✅ Settings are applied correctly
- ✅ Bucket appears in buckets list

### Test Case 7.2: Upload File

```bash
POST {{baseUrl}}/api/files/{{projectId}}/upload
Authorization: Bearer {{token}}
Content-Type: multipart/form-data

Form Data:
- file: [select a test image file]
- bucket: test-bucket
- isPublic: false

# Expected Response: 201 Created
# Save fileId from response
```

**Validation**:
- ✅ File is uploaded successfully
- ✅ File metadata is stored correctly
- ✅ File URL is returned
- ✅ File can be accessed via URL

### Test Case 7.3: Download File

```bash
GET {{baseUrl}}/api/files/{{fileId}}/download
Authorization: Bearer {{token}}

# Expected Response: 200 OK (file stream)
```

**Validation**:
- ✅ File is downloaded correctly
- ✅ Content-Type header is correct
- ✅ File size matches uploaded file

### Test Case 7.4: List Files

```bash
GET {{baseUrl}}/api/files/{{projectId}}?bucket=test-bucket&limit=10
Authorization: Bearer {{token}}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Files are listed correctly
- ✅ Filtering by bucket works
- ✅ Pagination works

### Test Case 7.5: Delete File

```bash
DELETE {{baseUrl}}/api/files/{{fileId}}
Authorization: Bearer {{token}}

# Expected Response: 200 OK
```

**Validation**:
- ✅ File is deleted successfully
- ✅ File cannot be accessed after deletion
- ✅ Storage space is freed

---

## Real-time Features Testing

### Test Case 8.1: WebSocket Connection

```javascript
// Using browser console or WebSocket client
const socket = io('http://localhost:5000', {
  auth: {
    token: '{{token}}'
  }
});

socket.on('connect', () => {
  console.log('Connected:', socket.id);
});

// Expected: Connection successful
```

**Validation**:
- ✅ Connection is established
- ✅ Authentication token is accepted
- ✅ Connection ID is returned

### Test Case 8.2: Subscribe to Project Events

```javascript
socket.emit('join:project', {
  projectId: '{{projectId}}'
});

socket.on('database:{{collectionSlug}}:create', (data) => {
  console.log('Document created:', data);
});

// Create a document and verify event is received
```

**Validation**:
- ✅ Subscription is successful
- ✅ Events are received in real-time
- ✅ Event data is complete

### Test Case 8.3: Real-time Updates

```javascript
// In one browser tab: Subscribe to updates
socket.emit('join:project', { projectId: '{{projectId}}' });
socket.on('database:{{collectionSlug}}:update', (data) => {
  console.log('Document updated:', data);
});

// In another tab/request: Update document
// Expected: Update event is received in first tab
```

**Validation**:
- ✅ Updates are broadcasted in real-time
- ✅ All subscribers receive updates
- ✅ Update data is accurate

---

## Billing & Payment Testing

### Test Case 9.1: Create Subscription Plan

```bash
POST {{baseUrl}}/api/billing/plans
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "Test Plan",
  "amount": 2900,
  "currency": "usd",
  "interval": "month",
  "features": {
    "apiCalls": 10000,
    "storage": 1000000
  }
}

# Expected Response: 201 Created
# Save planId from response
```

**Validation**:
- ✅ Plan is created successfully
- ✅ Plan details are correct
- ✅ Plan appears in plans list

### Test Case 9.2: Create Subscription

```bash
POST {{baseUrl}}/api/billing/subscriptions
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "planId": "{{planId}}",
  "paymentMethod": "card"
}

# Expected Response: 201 Created
# Save subscriptionId from response
```

**Validation**:
- ✅ Subscription is created
- ✅ Payment is processed (test mode)
- ✅ Subscription status is active

### Test Case 9.3: View Invoices

```bash
GET {{baseUrl}}/api/billing/projects/{{projectId}}/invoices
Authorization: Bearer {{token}}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Invoices are listed
- ✅ Invoice details are correct
- ✅ PDF download URL is available

---

## Wallet Service Testing

### Test Case 10.1: Create Wallet

```bash
POST {{baseUrl}}/api/wallet/projects/{{projectId}}/wallets
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "currency": "BTC",
  "label": "Test BTC Wallet"
}

# Expected Response: 201 Created
# Save walletId from response
```

**Validation**:
- ✅ Wallet is created successfully
- ✅ Address is generated
- ✅ Wallet is encrypted
- ✅ Wallet appears in wallets list

### Test Case 10.2: Get Wallet Balance

```bash
GET {{baseUrl}}/api/wallet/projects/{{projectId}}/wallets/{{walletId}}/balance
Authorization: Bearer {{token}}

# Expected Response: 200 OK
```

**Validation**- ✅ Balance is returned correctly
- ✅ Balance updates after transactions

### Test Case 10.3: Get Transaction History

```bash
GET {{baseUrl}}/api/wallet/projects/{{projectId}}/wallets/{{walletId}}/transactions
Authorization: Bearer {{token}}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Transactions are listed
- ✅ Transaction details are accurate
- ✅ Pagination works

---

## Integration & Webhooks Testing

### Test Case 11.1: Create Webhook

```bash
POST {{baseUrl}}/api/projects/{{projectId}}/webhooks
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "url": "https://webhook.site/your-unique-url",
  "events": ["data.create", "data.update", "data.delete"],
  "secret": "webhook-secret-key"
}

# Expected Response: 201 Created
# Save webhookId from response
```

**Validation**:
- ✅ Webhook is created successfully
- ✅ Webhook appears in webhooks list
- ✅ Events are configured correctly

### Test Case 11.2: Trigger Webhook

```bash
# Create a document (should trigger webhook)
POST {{baseUrl}}/api/projects/{{projectId}}/collections/{{usersCollectionId}}/data
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "email": "webhook-test@example.com",
  "name": "Webhook Test User"
}

# Check webhook.site or your webhook endpoint
# Expected: Webhook payload received
```

**Validation**:
- ✅ Webhook is triggered
- ✅ Payload is correct
- ✅ Signature is valid
- ✅ Retry logic works on failure

### Test Case 11.3: View Webhook Logs

```bash
GET {{baseUrl}}/api/projects/{{projectId}}/webhooks/{{webhookId}}/logs
Authorization: Bearer {{token}}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Logs are available
- ✅ Log details are complete
- ✅ Status codes are accurate

---

## Chat System Testing

### Test Case 12.1: Create Chat

```bash
POST {{baseUrl}}/api/projects/{{projectId}}/chats
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "Test Chat",
  "type": "group",
  "participants": ["{{userId1}}", "{{userId2}}"],
  "description": "Test group chat"
}

# Expected Response: 201 Created
# Save chatId from response
```

**Validation**:
- ✅ Chat is created successfully
- ✅ Participants are added
- ✅ Chat appears in chats list

### Test Case 12.2: Send Message

```bash
POST {{baseUrl}}/api/projects/{{projectId}}/chats/{{chatId}}/messages
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "type": "text",
  "content": "Hello, this is a test message!"
}

# Expected Response: 201 Created
# Save messageId from response
```

**Validation**:
- ✅ Message is sent successfully
- ✅ Message appears in chat
- ✅ Real-time delivery works

### Test Case 12.3: Get Chat Messages

```bash
GET {{baseUrl}}/api/projects/{{projectId}}/chats/{{chatId}}/messages?limit=50
Authorization: Bearer {{token}}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Messages are retrieved
- ✅ Pagination works
- ✅ Messages are ordered correctly

---

## Search Functionality Testing

### Test Case 13.1: Index Collection

```bash
POST {{baseUrl}}/api/projects/{{projectId}}/search/index/{{usersCollectionId}}
Authorization: Bearer {{token}}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Collection is indexed successfully
- ✅ Indexing completes without errors

### Test Case 13.2: Search Documents

```bash
GET {{baseUrl}}/api/projects/{{projectId}}/search?q=test&collection={{usersCollectionId}}
Authorization: Bearer {{token}}

# Expected Response: 200 OK
```

**Validation**:
- ✅ Search returns relevant results
- ✅ Results are ranked correctly
- ✅ Highlighting works (if supported)

---

## Multi-Vendor Application Scenario

This is a comprehensive end-to-end test simulating a real food delivery application.

### Step 1: Setup Multi-Role Feature

```bash
# Initialize multi-role feature
GET {{baseUrl}}/api/projects/{{projectId}}/multi-role
Authorization: Bearer {{token}}

# Configure vendor role
PATCH {{baseUrl}}/api/projects/{{projectId}}/multi-role/roles/vendor
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "requiresApproval": false,
  "requiresPayment": true,
  "requiresKYC": false
}

# Configure rider role
PATCH {{baseUrl}}/api/projects/{{projectId}}/multi-role/roles/rider
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "requiresApproval": false,
  "requiresKYC": true
}
```

### Step 2: Create Collections

```bash
# Vendors Collection
POST {{baseUrl}}/api/projects/{{projectId}}/collections
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "vendors",
  "slug": "vendors",
  "fields": [
    {"name": "businessName", "type": "string", "required": true},
    {"name": "ownerId", "type": "string", "required": true},
    {"name": "status", "type": "string", "default": "active"}
  ],
  "permissions": [
    {
      "role": "vendor",
      "actions": ["read", "update"],
      "conditions": {"ownerId": "$userId"}
    }
  ]
}

# Products Collection
POST {{baseUrl}}/api/projects/{{projectId}}/collections
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "products",
  "slug": "products",
  "fields": [
    {"name": "name", "type": "string", "required": true},
    {"name": "price", "type": "number", "required": true},
    {"name": "vendorId", "type": "string", "required": true},
    {"name": "status", "type": "string", "default": "active"}
  ],
  "permissions": [
    {
      "role": "vendor",
      "actions": ["create", "read", "update", "delete"],
      "conditions": {"vendorId": "$user.metadata.vendorId"}
    },
    {
      "role": "customer",
      "actions": ["read"]
    }
  ]
}

# Orders Collection
POST {{baseUrl}}/api/projects/{{projectId}}/collections
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "orders",
  "slug": "orders",
  "fields": [
    {"name": "customerId", "type": "string", "required": true},
    {"name": "vendorId", "type": "string", "required": true},
    {"name": "riderId", "type": "string", "required": false},
    {"name": "items", "type": "array", "required": true},
    {"name": "total", "type": "number", "required": true},
    {"name": "status", "type": "string", "default": "pending"}
  ],
  "permissions": [
    {
      "role": "customer",
      "actions": ["create", "read", "update"],
      "conditions": {"customerId": "$userId"}
    },
    {
      "role": "vendor",
      "actions": ["read", "update"],
      "conditions": {"vendorId": "$user.metadata.vendorId"}
    },
    {
      "role": "rider",
      "actions": ["read", "update"],
      "conditions": {"riderId": "$userId"}
    }
  ]
}
```

### Step 3: User Registration Flow

```bash
# Register Customer
POST {{baseUrl}}/api/auth/local/signup/customer
Content-Type: application/json

{
  "email": "customer@example.com",
  "password": "CustomerPass123!",
  "firstName": "John",
  "lastName": "Customer",
  "projectId": "{{projectId}}"
}
# Save customerToken

# Register Vendor
POST {{baseUrl}}/api/auth/local/signup/vendor
Content-Type: application/json

{
  "email": "vendor@restaurant.com",
  "password": "VendorPass123!",
  "firstName": "Jane",
  "lastName": "Vendor",
  "projectId": "{{projectId}}"
}
# Save vendorToken and vendorUserId

# Register Rider
POST {{baseUrl}}/api/auth/local/signup/rider
Content-Type: application/json

{
  "email": "rider@delivery.com",
  "password": "RiderPass123!",
  "firstName": "Mike",
  "lastName": "Rider",
  "projectId": "{{projectId}}"
}
# Save riderToken and riderUserId
```

### Step 4: Vendor Operations

```bash
# Create Vendor Record
POST {{baseUrl}}/api/projects/{{projectId}}/collections/{{vendorsCollectionId}}/data
Authorization: Bearer {{vendorToken}}
Content-Type: application/json

{
  "businessName": "Pizza Palace",
  "ownerId": "{{vendorUserId}}",
  "status": "active"
}
# Save vendorId

# Create Product (as vendor)
POST {{baseUrl}}/api/projects/{{projectId}}/collections/{{productsCollectionId}}/data
Authorization: Bearer {{vendorToken}}
Content-Type: application/json

{
  "name": "Margherita Pizza",
  "price": 15.99,
  "vendorId": "{{vendorId}}",
  "status": "active"
}
# Save productId
```

### Step 5: Customer Operations

```bash
# Browse Products (as customer)
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{productsCollectionId}}/data
Authorization: Bearer {{customerToken}}

# Create Order (as customer)
POST {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data
Authorization: Bearer {{customerToken}}
Content-Type: application/json

{
  "customerId": "{{customerUserId}}",
  "vendorId": "{{vendorId}}",
  "items": [
    {
      "productId": "{{productId}}",
      "quantity": 2,
      "price": 15.99
    }
  ],
  "total": 31.98,
  "status": "pending"
}
# Save orderId
```

### Step 6: Vendor Fulfillment

```bash
# View Orders (as vendor)
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data
Authorization: Bearer {{vendorToken}}

# Update Order Status (as vendor)
PATCH {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data/{{orderId}}
Authorization: Bearer {{vendorToken}}
Content-Type: application/json

{
  "status": "ready"
}
```

### Step 7: Rider Operations

```bash
# View Available Orders (as rider)
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data?filter={"status":"ready","riderId":null}
Authorization: Bearer {{riderToken}}

# Accept Order (as rider)
PATCH {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data/{{orderId}}
Authorization: Bearer {{riderToken}}
Content-Type: application/json

{
  "riderId": "{{riderUserId}}",
  "status": "assigned"
}

# Update Delivery Status (as rider)
PATCH {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data/{{orderId}}
Authorization: Bearer {{riderToken}}
Content-Type: application/json

{
  "status": "delivered"
}
```

### Step 8: Verify Permissions

**Test Case**: Customer cannot create products
```bash
POST {{baseUrl}}/api/projects/{{projectId}}/collections/{{productsCollectionId}}/data
Authorization: Bearer {{customerToken}}
Content-Type: application/json

{
  "name": "Unauthorized Product",
  "price": 10.00,
  "vendorId": "{{vendorId}}"
}

# Expected Response: 403 Forbidden
```

**Test Case**: Vendor cannot see other vendor's products
```bash
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{productsCollectionId}}/data
Authorization: Bearer {{vendorToken}}

# Expected Response: Only vendor's own products
```

---

## Test Checklist

Use this checklist to ensure all functionalities are tested:

### Authentication & Authorization
- [ ] Local registration
- [ ] Email verification
- [ ] Login/Logout
- [ ] Password reset
- [ ] Token refresh
- [ ] Role-based signup (customer, vendor, rider)
- [ ] OAuth authentication (if configured)

### Multi-Role Feature
- [ ] Initialize multi-role feature
- [ ] Get available roles
- [ ] Toggle roles on/off
- [ ] Update role configuration
- [ ] Add custom role
- [ ] Configure collection permissions
- [ ] Role-based signup endpoints

### Database Operations
- [ ] Create collections
- [ ] Create documents
- [ ] Read documents (single & list)
- [ ] Update documents
- [ ] Delete documents
- [ ] Filter and search
- [ ] Pagination
- [ ] Sorting

### File Storage
- [ ] Create buckets
- [ ] Upload files
- [ ] Download files
- [ ] List files
- [ ] Delete files
- [ ] File permissions

### Real-time Features
- [ ] WebSocket connection
- [ ] Subscribe to events
- [ ] Receive real-time updates
- [ ] Chat system
- [ ] Presence indicators

### Billing & Payments
- [ ] Create plans
- [ ] Create subscriptions
- [ ] View invoices
- [ ] Download invoice PDF

### Wallet Service
- [ ] Create wallets
- [ ] Get balances
- [ ] View transactions
- [ ] Send transactions (testnet)

### Integrations
- [ ] Create webhooks
- [ ] Webhook delivery
- [ ] Webhook retries
- [ ] View webhook logs

### Search
- [ ] Index collections
- [ ] Search documents
- [ ] Search results ranking

### Multi-Vendor Scenario
- [ ] Complete order flow
- [ ] Role-based access control
- [ ] Permission enforcement
- [ ] Real-time order updates

---

## Reporting Issues

When reporting bugs, include:

1. **Test Case ID**: e.g., "Test Case 5.3"
2. **Expected Behavior**: What should happen
3. **Actual Behavior**: What actually happened
4. **Steps to Reproduce**: Detailed steps
5. **Request/Response**: Full API request and response (remove sensitive data)
6. **Environment**: Development/Staging/Production
7. **Screenshots/Logs**: If applicable

---

## Next Steps

After completing functional testing, proceed to:
- **[Security Testing Guide](./TESTING_GUIDE_SECURITY.md)** - Test for vulnerabilities like IDOR, privilege escalation, access control issues, race conditions, etc.

---

**Last Updated**: December 2024  
**Version**: 1.0.0

