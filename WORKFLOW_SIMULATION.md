# MUDBASE Backend-as-a-Service: Complete Workflow Simulation

> **📅 Last Updated**: December 2024  
> **✨ NEW FEATURES**: See [NEW_FEATURES_WORKFLOWS.md](./NEW_FEATURES_WORKFLOWS.md) for complete documentation of 24+ new features added in December 2024!

## Quick Links

- **🆕 [New Features Documentation](./NEW_FEATURES_WORKFLOWS.md)** - All December 2024 features
- **🎯 [RBAC Guide](./RBAC_GUIDE.md)** - Multi-role & custom permissions (E-commerce, Delivery Apps, SaaS)
- **📊 [Competitive Comparison](./COMPETITIVE_COMPARISON.md)** - Updated platform scores
- **📋 [Completion Report](./COMPLETION_REPORT.md)** - Comprehensive improvement summary
- **🔐 [Compliance Roadmap](./COMPLIANCE_ROADMAP.md)** - SOC 2, GDPR, HIPAA plans
- **💰 [Wallet Roadmap](./WALLET_ROADMAP.md)** - Future wallet features
- **🔍 [Full-Text Search Guide](./FULL_TEXT_SEARCH.md)** - Search documentation

## ⭐ Multi-Role Applications Supported!

**MUDBASE fully supports complex multi-role applications:**
- ✅ E-commerce (customers, sellers, admins, support)
- ✅ Delivery apps (riders, customers, restaurants, dispatchers)
- ✅ SaaS platforms (org owners, admins, team members, guests)
- ✅ Marketplaces (buyers, sellers, moderators)
- ✅ Healthcare (patients, doctors, nurses)
- ✅ **ANY multi-role application**

**🆕 NEW: Multi-Role Feature - Ready-to-Use System!**
- ✅ Pre-built role templates (Owner, Admin, Support, Seller, Vendor, Rider, Customer)
- ✅ Role-based signup endpoints (e.g., `/signup/customer`, `/signup/vendor`)
- ✅ Dashboard configuration - no coding required
- ✅ Works with all authentication methods
- ✅ See [MULTI_ROLE_FEATURE.md](./MULTI_ROLE_FEATURE.md) for complete guide

**See [RBAC_GUIDE.md](./RBAC_GUIDE.md) for complete documentation!**

## Table of Contents
1. [System Architecture Overview](#system-architecture-overview)
2. [Authentication Flows](#authentication-flows)
   - 2.1. [Anonymous Authentication (NEW)](#anonymous-authentication-new)
   - 2.2. [OAuth Account Linking (NEW)](#oauth-account-linking-new)
   - 2.3. [GDPR Compliance (NEW)](#gdpr-compliance-new)
3. [Organization & Project Setup](#organization--project-setup)
4. [Wallet as a Service Workflows](#wallet-as-a-service-workflows)
5. [Billing & Payment Gateway Workflows](#billing--payment-gateway-workflows)
   - 5.1. [Stripe Invoicing (NEW)](#stripe-invoicing-new)
6. [Database Operations](#database-operations)
   - 6.1. [Row-Level Security (RLS) (NEW)](#row-level-security-rls-new)
   - 6.2. [Backup & Restore (NEW)](#backup--restore-new)
7. [File Storage](#file-storage)
   - 7.1. [Virus Scanning (NEW)](#virus-scanning-new)
   - 7.2. [Per-File RBAC (NEW)](#per-file-rbac-new)
8. [Real-time Features](#real-time-features)
   - 8.1. [Real-time Analytics (NEW)](#real-time-analytics-new)
   - 8.2. [Chat System](#chat-system)
9. [Integration System](#integration-system)
   - 9.1. [Webhook Transformations (NEW)](#webhook-transformations-new)
10. [Compliance & Security (NEW)](#compliance--security-new)
11. [Monitoring & Tracing (NEW)](#monitoring--tracing-new)
12. [Complete End-to-End Scenarios](#complete-end-to-end-scenarios)
13. [API Reference Quick Guide](#api-reference-quick-guide)

---

## System Architecture Overview

### Core Components

```
┌────────────────────────────────────────────────────────────┐
│                    MUDBASE Backend Platform                │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Auth Layer  │  │  API Layer   │  │  Socket.IO   │      │
│  │              │  │              │  │  Real-time   │      │
│  │ • Local      │  │ • REST API   │  │ • Chat       │      │
│  │ • OAuth      │  │ • Webhooks   │  │ • Events     │      │
│  │ • Magic Link │  │ • GraphQL    │  │ • Database   │      │
│  │ • OTP        │  │              │  │              │      │
│  │ • 2FA        │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Database   │  │   Storage    │  │  Integrations│      │
│  │              │  │              │  │              │      │
│  │ • MongoDB    │  │ • File Store │  │ • 50+ APIs   │      │
│  │ • Collections│  │ • Buckets    │  │ • Webhooks   │      │
│  │ • Indexes    │  │ • CDN        │  │ • Custom     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Wallet     │  │   Billing    │  │  Security    │      │
│  │   Service    │  │              │  │              │      │
│  │              │  │ • Paystack   │  │ • RBAC       │      │
│  │ • BTC        │  │ • Flutterwave│  │ • Encryption │      │
│  │ • ETH        │  │ • Plans      │  │ • Rate Limit│       │
│  │ • SOL        │  │ • Usage      │  │ • Audit Log  │      │
│  │ • TRX        │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└────────────────────────────────────────────────────────────┘
```

### Data Flow

```
User Request → Authentication → Authorization (RBAC) → Business Logic → Database/External API → Response
                ↓
            JWT Token
                ↓
        Session Management (30min idle timeout)
                ↓
        Real-time Updates via Socket.IO
```

---

## Authentication Flows

### 1. Basic Authentication (Organization-based)

These endpoints are for organization-level authentication and create organizations automatically.

#### Registration Flow

```javascript
// Step 1: User Registration (creates organization automatically)
POST /api/auth/register
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "firstName": "John",
  "lastName": "Doe",
  "orgName": "Acme Corp" // Optional
}

// Response
{
  "message": "User registered successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner",
    "emailVerified": false,
    "org": "507f1f77bcf86cd799439013"
  }
}

// Step 2: Login
POST /api/auth/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]"
}

// Response
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  }
}

// Step 3: Get Session
GET /api/auth/session
Authorization: Bearer {token}

// Response
{
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner"
  },
  "authenticated": true
}

// Step 4: Password Reset Request
POST /api/auth/password-reset
Content-Type: application/json

{
  "email": "john.doe@example.com"
}

// Response
{
  "message": "Password reset email sent"
}

// Step 5: Reset Password
POST /api/auth/password-reset/{token}
Content-Type: application/json

{
  "password": "[NEW_PASSWORD]"
}

// Response
{
  "message": "Password reset successful"
}
```

### 2. Local Authentication (Project-based)

These endpoints are for project-level authentication and require a projectId.

#### Registration Flow

```javascript
// Step 1: User Registration
POST /api/auth/local/register
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "firstName": "John",
  "lastName": "Doe",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "message": "User registered successfully",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "emailVerified": false,
    "role": "developer",
    "org": "507f1f77bcf86cd799439013"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

// Step 2: Email Verification (Optional)
POST /api/auth/verify-email
Authorization: Bearer {token}
Content-Type: application/json

{
  "token": "email-verification-token-from-email"
}

// Response
{
  "success": true,
  "message": "Email verified successfully"
}

// Step 3: Login
POST /api/auth/local/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  },
  "expiresIn": "24h"
}
```

#### Password Reset (Project-based)

```javascript
// Step 1: Request password reset
POST /api/auth/local/password-reset
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "projectId": "507f1f77bcf86cd799439011" // Optional
}

// Response
{
  "message": "Password reset email sent"
}

// Step 2: Reset password
POST /api/auth/local/password-reset/{token}
Content-Type: application/json

{
  "password": "[NEW_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011" // Optional
}

// Response
{
  "message": "Password reset successful"
}
```

#### Session Management

```javascript
// Session expires after 30 minutes of inactivity
// Server automatically extends session on activity (rolling: true)

// Check current session (organization-based)
GET /api/auth/session
Authorization: Bearer {token}

// Check current session (project-based)
GET /api/auth/local/session?projectId={projectId}
Authorization: Bearer {token}

// Response
{
  "user": {
    "id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "project": {
      "id": "507f1f77bcf86cd799439011",
      "name": "My Awesome App",
      "role": "developer"
    }
  },
  "authenticated": true
}

// Response
{
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  }
}

// Refresh token (if implemented)
POST /api/auth/refresh
Authorization: Bearer {token}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": "24h"
}

// Logout (organization-based)
POST /api/auth/logout
Authorization: Bearer {token}

// Response
{
  "message": "Logout successful"
}

// Logout (project-based)
POST /api/auth/local/logout
Authorization: Bearer {token}

// Response
{
  "message": "Logout successful"
}
```

**Key Differences Between Organization-based and Project-based Auth:**

- **Organization-based** (`/api/auth/*`): Creates organizations automatically, simpler flow, good for admin/owner accounts
- **Project-based** (`/api/auth/local/*`): Requires existing project, project-specific authentication, includes rate limiting and captcha verification, better for end-user authentication

### 3. OAuth Authentication

#### Supported Providers
- Google, GitHub, Facebook, Microsoft, Apple, LinkedIn, Discord, Twitter, etc.

#### OAuth Flow Example (Google)

```javascript
// Step 1: Initiate OAuth
// Redirect user to:
GET /api/auth/oauth/google?projectId=507f1f77bcf86cd799439011

// User is redirected to Google consent screen
// After consent, Google redirects to:
GET /api/auth/oauth/google/callback?code={authorization_code}

// Step 2: Backend processes OAuth callback
// Backend automatically:
// 1. Exchanges code for access token
// 2. Fetches user profile from Google
// 3. Creates/updates user in database
// 4. Generates JWT token
// 5. Redirects to frontend with token

// Frontend receives redirect:
https://yourapp.com/auth/callback?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// Step 3: Use token for authenticated requests
Authorization: Bearer {token}

// Response (from OAuth callback redirect)
// Frontend receives:
https://yourapp.com/auth/callback?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...&user={"id":"507f1f77bcf86cd799439012","email":"john.doe@example.com"}
```

#### OAuth Provider Configuration

```javascript
// Configure OAuth provider in project settings
PATCH /api/projects/{projectId}
Authorization: Bearer {admin_token}
Content-Type: application/json

{
  "authProviders": {
    "google": {
      "enabled": true,
      "clientId": "your-google-client-id",
      "clientSecret": "your-google-client-secret",
      "callbackUrl": "https://api.mudbase.com/api/auth/oauth/google/callback"
    },
    "github": {
      "enabled": true,
      "clientId": "your-github-client-id",
      "clientSecret": "your-github-client-secret"
    }
  }
}
```

### 4. Magic Link Authentication

```javascript
// Step 1: Request Magic Link
POST /api/auth/magic-link/send
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "projectId": "507f1f77bcf86cd799439011",
  "redirectUrl": "https://yourapp.com/auth/verify"
}

// Response
{
  "success": true,
  "message": "Magic link sent to email"
}

// Step 2: User clicks link in email
// Email contains: https://api.mudbase.com/api/auth/magic-link/verify?token={magic_token}

// Step 3: Verify Magic Link
GET /api/auth/magic-link/verify?token={magic_token}&projectId={projectId}

// Response (redirects to frontend with token)
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

### 5. OTP Authentication (SMS/Email)

```javascript
// Step 1: Request OTP
POST /api/auth/otp/send
Content-Type: application/json

{
  "identifier": "john.doe@example.com", // or phone number
  "method": "email", // or "sms" or "auto"
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "message": "OTP sent to email",
  "expiresIn": 300 // 5 minutes
}

// Step 2: Verify OTP
POST /api/auth/otp/verify
Content-Type: application/json

{
  "identifier": "john.doe@example.com",
  "otp": "123456",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

### 6. Two-Factor Authentication (2FA)

```javascript
// Step 1: Enable 2FA
POST /api/auth/2fa/setup
Authorization: Bearer {token}
Content-Type: application/json

// Response
{
  "secret": "JBSWY3DPEHPK3PXP",
  "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..."
}

// Step 2: Verify and Enable
POST /api/auth/2fa/verify
Authorization: Bearer {token}
Content-Type: application/json

{
  "token": "123456" // TOTP code from authenticator app
}

// Step 3: Login with 2FA
POST /api/auth/local/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response (requires 2FA)
{
  "requires2FA": true,
  "tempToken": "temp-token-for-2fa-verification"
}

// Step 4: Verify 2FA
POST /api/auth/2fa/verify-login
Content-Type: application/json

{
  "tempToken": "temp-token-for-2fa-verification",
  "token": "123456"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

---

## Organization & Project Setup

### Complete Setup Workflow

```javascript
// Step 1: Create Organization
POST /api/orgs
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Acme Corporation",
  "slug": "acme-corp" // auto-generated if not provided
}

// Response
{
  "success": true,
  "org": {
    "_id": "507f1f77bcf86cd799439013",
    "name": "Acme Corporation",
    "slug": "acme-corp",
    "members": ["507f1f77bcf86cd799439012"],
    "createdAt": "2024-01-15T10:00:00Z"
  }
}

// Step 2: Create Project
POST /api/projects
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "My Awesome App",
  "orgId": "507f1f77bcf86cd799439013",
  "description": "A revolutionary app"
}

// Response
{
  "success": true,
  "project": {
    "_id": "507f1f77bcf86cd799439011",
    "name": "My Awesome App",
    "slug": "my-awesome-app",
    "org": "507f1f77bcf86cd799439013",
    "apiKey": "pk_live_abc123...",
    // NOTE: secretKey is NEVER returned in responses
    "settings": {
      "auth": {
        "requireEmailVerification": true,
        "sessionTimeout": 1800000
      }
    }
  }
}

// Step 3: Configure Project Settings
PATCH /api/projects/507f1f77bcf86cd799439011
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "settings": {
    "auth": {
      "providers": {
        "local": { "enabled": true },
        "google": { "enabled": true },
        "github": { "enabled": true },
        "magicLink": { "enabled": true },
        "otp": { "enabled": true }
      },
      "requireEmailVerification": true,
      "sessionTimeout": 1800000,
      "passwordPolicy": {
        "minLength": 8,
        "requireUppercase": true,
        "requireNumbers": true,
        "requireSpecialChars": true
      }
    },
    "database": {
      "collections": {
        "users": { "enabled": true },
        "products": { "enabled": true },
        "orders": { "enabled": true }
      }
    },
    "storage": {
      "maxFileSize": 10485760, // 10MB
      "allowedTypes": ["image/jpeg", "image/png", "application/pdf"]
    }
  }
}

// Step 4: Set up Payment Gateway (for billing)
POST /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "provider": "paystack",
  "publicKey": "pk_test_abc123...",
  "secretKey": "[SECRET_KEY_NOT_RETURNED]", // Only sent during creation, never returned
  "config": {
    "currency": "NGN",
    "webhookUrl": "https://api.mudbase.com/api/billing/webhooks/paystack"
  }
}

// Response
{
  "message": "Payment gateway created successfully",
  "gateway": {
    "_id": "507f1f77bcf86cd799439014",
    "provider": "paystack",
    "status": "pending"
    // NOTE: secretKey and webhookSecret are NEVER returned in responses
  }
}

// Step 5: Activate Payment Gateway
PATCH /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "status": "active"
}
```

---

## Wallet as a Service Workflows

### Important Security Notes

1. **Private Key Generation**: Each wallet gets its own unique private key. Keys are generated per wallet, not shared per project or organization.
2. **Private Key Visibility**: Private keys are ONLY shown once in the `/api/wallet/generate-key` endpoint response. They are NEVER returned in:
   - Wallet creation responses
   - Wallet listing responses
   - Wallet balance responses
   - Any other wallet-related endpoints
3. **Encryption**: All private keys are encrypted using AES-256-GCM before storage in the database.
4. **Key Storage**: Users must save the private key securely when generated. It cannot be retrieved later.

### Understanding Wallet Encryption

**Two Types of Keys**:

1. **Project Encryption Key** (`walletEncryptionKey`):
   - One per project (auto-generated)
   - Used to encrypt/decrypt wallet private keys for database storage
   - Provides additional security layer and project isolation
   - **NOT used for signing transactions**

2. **Wallet Private Keys**:
   - One unique key per wallet (generated per wallet)
   - The actual cryptocurrency private key
   - Used to sign blockchain transactions
   - Encrypted using project encryption key before storage

**Transaction Flow**:
- When making a transaction, the wallet's private key is decrypted using the project encryption key
- The decrypted wallet private key is then used to sign the transaction
- The project encryption key is NEVER used to sign transactions - only the wallet's private key

For detailed explanation, see `docs/WALLET_ENCRYPTION_EXPLAINED.md`.

### Supported Cryptocurrencies
- **BTC** (Bitcoin) - Bech32 addresses (bc1...)
- **ETH** (Ethereum)
- **BNB** (Binance Smart Chain)
- **SOL** (Solana)
- **TRX** (Tron)
- **LTC** (Litecoin)
- **USDT** (Tether - Multi-network: ETH, TRX, BSC, SOL, POLYGON)

### Platform Fee Structure (Hybrid Model)

**Formula**: `Platform Fee = max(1% * amount, minimumFee)`

| Currency | Minimum Fee | Minimum Withdrawal | Notes |
|----------|-------------|-------------------|-------|
| BTC | 0.00012 BTC (~$8) | 0.0002 BTC (~$13) | Updated based on market rates |
| ETH | 0.001 ETH (~$2.50) | 0.001 ETH (~$2.50) | Increased to protect against gas spikes |
| BNB | 0.0002 BNB (~$0.13) | 0.001 BNB (~$0.65) | Updated based on market rates |
| SOL | 0.008 SOL (~$1.20) | 0.01 SOL (~$1.50) | Platform as feePayer (no pre-fund) |
| TRX | 1 TRX (~$0.05) | 5 TRX (~$0.25) | Lowered for micro-transactions support |
| LTC | 0.0001 LTC (~$0.01) | 0.001 LTC (~$0.10) | Updated based on market rates |
| USDT-ETH | $4 fixed | 5 USDT | Covers gas spikes |
| USDT-BSC | $0.75 fixed | 2 USDT | Scaled with network cost + profit margin |
| USDT-TRX | $0.50 fixed | 2 USDT | Scaled with network cost + profit margin |
| USDT-SOL | $0.15 fixed | 2 USDT | Scaled with network cost + profit margin |

**Profitability Guardrails**: Transactions are rejected if platform fee < estimated network cost + pre-fund cost.

**Important Notes**:
- **ETH Gas Spikes**: ETH minimum fee (0.001 ETH) protects against gas spikes, but network conditions may still cause fees to exceed minimum during extreme congestion. Consider implementing dynamic fee adjustments for ETH during high gas periods.
- **USDT Fees**: Fees on non-ETH chains (BSC, TRX, SOL) are scaled with network cost + profit margin rather than flat rates for better user experience and competitiveness.
- **Micro-Transactions**: TRX minimum withdrawal lowered to 5 TRX to support small users and micro-transactions.

For detailed fee structure and rationale, see [FEE_STRUCTURE_UPDATED.md](./FEE_STRUCTURE_UPDATED.md)

### 1. Generate Private Key (Dashboard Endpoint)

**Important**: Each wallet gets its own unique private key. Keys are generated per wallet, not per project. The private key is only shown once in this response and must be saved securely.

```javascript
// Generate a new key pair for any supported currency
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "network": null // Only required for USDT
}

// Response
{
  "success": true,
  "data": {
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "privateKey": "L5EZftvrYaSudiozVRzTqLcHLNDoVn7H5HSfM9BAN6pMA5zoi3WZ",
    "publicKey": "02a1633cafcc01ebfb6d78e39f687a1f0995c62fc95f51ead10a02ee0be551b5fb"
  },
  "warning": "This private key is shown only once. Store it securely."
}

// For USDT with network
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "USDT",
  "network": "ETH" // Options: ETH, TRX, BSC, SOL, POLYGON
}

// Response
{
  "success": true,
  "data": {
    "currency": "USDT",
    "network": "ETH",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "privateKey": "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
    "publicKey": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
  },
  "warning": "This private key is shown only once. Store it securely."
}
```

### 2. Create Wallet with Generated Private Key

**Security Note**: Private keys are NEVER returned in wallet creation responses. They are encrypted and stored securely. Only the address and wallet metadata are returned.

```javascript
// User can set the generated private key from dashboard
POST /api/wallet/create
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "privateKey": "[PRIVATE_KEY_FROM_GENERATE_KEY_ENDPOINT]",
  "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh"
}

// Response
{
  "success": true,
  "message": "BTC wallet created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439015",
    "user": "507f1f77bcf86cd799439012",
    "org": "507f1f77bcf86cd799439013",
    "project": "507f1f77bcf86cd799439011",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "balance": "0",
    "isCustomKey": true,
    "isActive": true,
    "createdAt": "2024-01-15T10:00:00Z",
    "updatedAt": "2024-01-15T10:00:00Z"
  }
}
```

### 3. Create Wallet with Auto-Generated Key

```javascript
// Let system generate the key pair
POST /api/wallet/create
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "ETH"
}

// Response
{
  "success": true,
  "message": "ETH wallet created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439016",
    "user": "507f1f77bcf86cd799439012",
    "org": "507f1f77bcf86cd799439013",
    "project": "507f1f77bcf86cd799439011",
    "currency": "ETH",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "balance": "0",
    "isCustomKey": false,
    "isActive": true,
    "createdAt": "2024-01-15T10:00:00Z",
    "updatedAt": "2024-01-15T10:00:00Z"
  }
}
```

### 4. Get User Wallets

```javascript
// Get all wallets for authenticated user
GET /api/wallet
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": [
    {
      "_id": "507f1f77bcf86cd799439015",
      "currency": "BTC",
      "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
      "balance": "0.05",
      "isCustomKey": true,
      "isActive": true,
      "project": "507f1f77bcf86cd799439011",
      "createdAt": "2024-01-15T10:00:00Z"
    },
    {
      "_id": "507f1f77bcf86cd799439016",
      "currency": "ETH",
      "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
      "balance": "2.5",
      "isCustomKey": false,
      "isActive": true,
      "project": "507f1f77bcf86cd799439011",
      "createdAt": "2024-01-15T10:00:00Z"
    }
  ]
}
```

### 4.5. Get Wallet Private Key

**Security Warning**: This endpoint returns the decrypted private key. Use only when you need to export your wallet. The private key is sensitive and should be kept secure.

```javascript
// Get private key for a specific wallet
GET /api/wallet/{walletId}/private-key
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "walletId": "507f1f77bcf86cd799439015",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "privateKey": "L5EZftvrYaSudiozVRzTqLcHLNDoVn7H5HSfM9BAN6pMA5zoi3WZ",
    "isCustomKey": true
  },
  "warning": "Keep this private key secure and never share it. Anyone with access to this key can control your wallet.",
  "security": {
    "accessedAt": "2024-01-15T11:20:00Z",
    "accessedBy": "507f1f77bcf86cd799439012"
  }
}
```

**Access Control**:
- User can only retrieve private keys for wallets they own
- Wallet must belong to the user's organization
- All access is logged for security audit

### 6. Get Wallet Balance

```javascript
// Get balance for specific wallet
GET /api/wallet/507f1f77bcf86cd799439015/balance
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "walletId": "507f1f77bcf86cd799439015",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "balance": "0.05",
    "balanceInUSD": 2500.00,
    "lastSyncedAt": "2024-01-15T10:30:00Z"
  }
}
```

### 7. Withdraw Funds (Async Processing)

**Important**: Withdrawals are processed asynchronously. The API returns immediately with a `queued` status. You must check the transaction status to see when it's confirmed. Real blockchain confirmations take time (BTC: 10-60 minutes, ETH: 15-45 seconds, SOL: 5-10 seconds, etc.).

**Fee Model**: Platform fees use a hybrid model: `max(1% * amount, minimumFee)`. Project fees (optional) are added on top if configured. Minimum fees are enforced per currency to ensure profitability.

```javascript
// Withdraw from wallet
POST /api/wallet/507f1f77bcf86cd799439015/withdraw
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "toAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "amount": 0.01,
  "feeRate": 10 // For BTC/LTC only
}

// Response (Returns Immediately - Async Processing)
{
  "success": true,
  "message": "Withdrawal is processing",
  "data": {
    "transactionId": "507f1f77bcf86cd799439027",
    "status": "queued",
    "amount": 0.01,
    "platformFee": 0.00012, // max(1% * 0.01, 0.00012 BTC minimum)
    "projectFee": 0.00005, // Optional: if project fee is configured
    "totalFee": 0.00017, // Platform fee + project fee
    "message": "Withdrawal is processing. Check transaction status for updates."
  }
}

// Check Transaction Status
GET /api/wallet/transactions/507f1f77bcf86cd799439027
Authorization: Bearer {user_token}

// Response (During Processing)
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439027",
    "status": "prefunding", // or "processing", "broadcasted"
    "mainTxStatus": "pending",
    "mainTxHash": null,
    "platformFee": 0.0005,
    "createdAt": "2024-01-15T10:35:00Z"
  }
}

// Response (After Confirmation)
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439027",
    "status": "confirmed",
    "mainTxHash": "abc123def456ghi789jkl012mno345pqr678stu901vwx234yz",
    "mainTxStatus": "confirmed",
    "refundTxHash": "xyz789abc123def456ghi789jkl012mno345pqr678",
    "refundStatus": "confirmed",
    "networkFee": 0.0001,
    "platformFee": 0.0005,
    "completedAt": "2024-01-15T10:45:00Z"
  }
}
```

**Transaction Status Flow**:
- `queued` → Transaction added to processing queue
- `prefunding` → Platform sending pre-fund (if needed)
- `processing` → User transaction being prepared
- `broadcasted` → Transaction broadcast to blockchain
- `confirmed` → Transaction confirmed on blockchain
- `completed` → All operations completed successfully
- `failed` → Transaction failed (check error field)
- `partial` → User tx succeeded but refund failed (CRITICAL - requires manual intervention)

### 8. Validate Address

```javascript
// Validate cryptocurrency address
POST /api/wallet/validate-address
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "currency": "BTC"
}

// Response
{
  "success": true,
  "data": {
    "isValid": true,
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "format": "bech32"
  }
}
```

### 9. Get Supported Currencies

```javascript
// Get list of supported currencies
GET /api/wallet/currencies
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": [
    {
      "code": "BTC",
      "name": "Bitcoin",
      "network": "mainnet"
    },
    {
      "code": "ETH",
      "name": "Ethereum",
      "network": "mainnet"
    },
    {
      "code": "BNB",
      "name": "Binance Coin",
      "network": "bsc"
    },
    {
      "code": "LTC",
      "name": "Litecoin",
      "network": "mainnet"
    },
    {
      "code": "SOL",
      "name": "Solana",
      "network": "mainnet"
    },
    {
      "code": "TRX",
      "name": "Tron",
      "network": "mainnet"
    },
    {
      "code": "USDT",
      "name": "Tether",
      "network": "ethereum"
    }
  ]
}
```

---

## Billing & Payment Gateway Workflows

### 1. Setup Payment Gateway

```javascript
// Create payment gateway account
POST /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "provider": "paystack", // or "flutterwave", "monnify", "interswitch"
  "publicKey": "pk_test_abc123...",
  "secretKey": "[SECRET_KEY_NOT_RETURNED]",
  "config": {
    "currency": "NGN",
    "webhookUrl": "https://api.mudbase.com/api/billing/webhooks/paystack"
  }
}

// Response
{
  "message": "Payment gateway created successfully",
  "gateway": {
    "_id": "507f1f77bcf86cd799439017",
    "provider": "paystack",
    "status": "pending",
    "publicKey": "pk_test_abc123...",
    "createdAt": "2024-01-15T10:40:00Z"
  }
}

// Activate gateway
PATCH /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "status": "active"
}
```

### 2. Create Subscription Plan

```javascript
// Create a plan for your project
POST /api/billing/projects/507f1f77bcf86cd799439011/plans
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Pro Plan",
  "description": "Professional plan with advanced features",
  "pricing": {
    "type": "recurring",
    "monthly": {
      "amount": 5000, // in kobo (50 NGN)
      "currency": "NGN"
    },
    "yearly": {
      "amount": 50000, // in kobo (500 NGN)
      "currency": "NGN"
    },
    "usage": [
      {
        "metric": "api_calls",
        "unitPrice": 0.1, // per API call
        "includedUnits": 10000,
        "currency": "NGN"
      }
    ]
  },
  "features": [
    {
      "name": "api_access",
      "description": "Full API access",
      "included": true
    },
    {
      "name": "storage",
      "description": "100GB storage",
      "included": true,
      "limit": 107374182400 // 100GB in bytes
    }
  ],
  "limits": {
    "apiCalls": 100000,
    "storage": 107374182400,
    "bandwidth": 1073741824000
  },
  "trial": {
    "enabled": true,
    "days": 7
  },
  "isActive": true,
  "isDefault": false
}

// Response
{
  "message": "Plan created successfully",
  "plan": {
    "_id": "507f1f77bcf86cd799439018",
    "name": "Pro Plan",
    "description": "Professional plan with advanced features",
    "pricing": {
      "type": "recurring",
      "monthly": {
        "amount": 5000,
        "currency": "NGN"
      },
      "yearly": {
        "amount": 50000,
        "currency": "NGN"
      }
    },
    "features": [
      {
        "name": "api_access",
        "description": "Full API access",
        "included": true
      },
      {
        "name": "storage",
        "description": "100GB storage",
        "included": true,
        "limit": 107374182400
      }
    ],
    "isActive": true,
    "createdAt": "2024-01-15T10:40:00Z"
  }
}
```

### 3. Customer Checkout Flow

```javascript
// Step 1: Get available plans (Public API)
GET /api/billing/public/projects/507f1f77bcf86cd799439011/plans

// Response
{
  "plans": [
    {
      "_id": "507f1f77bcf86cd799439018",
      "name": "Pro Plan",
      "description": "Professional plan with advanced features",
      "pricing": {
        "monthly": {
          "amount": 5000,
          "currency": "NGN"
        },
        "yearly": {
          "amount": 50000,
          "currency": "NGN"
        }
      },
      "features": [ ... ]
    }
  ]
}

// Step 2: Create checkout session
POST /api/billing/public/projects/507f1f77bcf86cd799439011/checkout
Content-Type: application/json

{
  "planId": "507f1f77bcf86cd799439018",
  "billingCycle": "monthly",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  },
  "successUrl": "https://yourapp.com/billing/success",
  "cancelUrl": "https://yourapp.com/billing/cancel"
}

// Response
{
  "success": true,
  "data": {
    "checkoutUrl": "https://paystack.com/pay/abc123...",
    "reference": "mudbase_1705315200_abc123def456",
    "amount": 5000,
    "currency": "NGN"
  }
}

// Step 3: Redirect customer to authorizationUrl
// Customer completes payment on Paystack/Flutterwave

// Step 4: Payment Gateway redirects to callback URL
// Backend automatically verifies payment and creates subscription

// Step 5: Verify payment (if needed manually)
POST /api/billing/public/projects/507f1f77bcf86cd799439011/verify-payment
Content-Type: application/json

{
  "reference": "mudbase_1705315200_abc123def456",
  "orgId": "507f1f77bcf86cd799439013"
}

// Response
{
  "success": true,
  "message": "Payment verified and subscription created",
  "data": {
    "subscription": {
      "_id": "507f1f77bcf86cd799439019",
      "status": "active",
      "plan": {
        "_id": "507f1f77bcf86cd799439018",
        "name": "Pro Plan"
      },
      "customerEmail": "customer@example.com",
      "currentPeriodEnd": "2024-02-15T10:45:00Z",
      "billingCycle": "monthly"
    }
  }
}
```

### 4. Check Subscription Status

```javascript
// Check customer subscription (Public API)
GET /api/billing/public/projects/507f1f77bcf86cd799439011/subscription?email=customer@example.com

// Response
{
  "hasSubscription": true,
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active",
    "plan": {
      "_id": "507f1f77bcf86cd799439018",
      "name": "Pro Plan",
      "price": 5000,
      "currency": "NGN"
    },
    "customerEmail": "customer@example.com",
    "currentPeriodEnd": "2024-02-15T10:45:00Z",
    "billingCycle": "monthly",
    "createdAt": "2024-01-15T10:45:00Z"
  }
}
```

### 5. Check Feature Access

```javascript
// Check if customer has access to a feature
GET /api/billing/public/projects/507f1f77bcf86cd799439011/feature-access?email=customer@example.com&feature=api_access

// Response
{
  "hasAccess": true,
  "reason": "Active subscription",
  "plan": {
    "_id": "507f1f77bcf86cd799439018",
    "name": "Pro Plan"
  },
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active"
  }
}
```

### 6. Record Usage (Metered Billing)

```javascript
// Record usage for metered billing
POST /api/billing/public/projects/507f1f77bcf86cd799439011/usage
Content-Type: application/json

{
  "email": "customer@example.com",
  "metric": "api_calls",
  "quantity": 150
}

// Response
{
  "success": true,
  "message": "Usage recorded successfully"
}
```

### 7. Cancel Subscription

```javascript
// Cancel subscription
POST /api/billing/subscriptions/507f1f77bcf86cd799439019/cancel
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "cancelImmediately": false // Cancel at period end
}

// Response
{
  "message": "Subscription canceled successfully",
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active",
    "cancelAtPeriodEnd": true,
    "currentPeriodEnd": "2024-02-15T10:45:00Z",
    "canceledAt": "2024-01-15T11:00:00Z"
  }
}
```

### 8. Payment Gateway Webhook

```javascript
// Payment gateway sends webhook on payment events
POST /api/billing/webhooks/paystack
Content-Type: application/json
X-Paystack-Signature: {signature}

{
  "event": "charge.success",
  "data": {
    "reference": "mudbase_1705315200_abc123def456",
    "amount": 5000,
    "currency": "NGN",
    "customer": {
      "email": "customer@example.com"
    },
    "metadata": {
      "projectId": "507f1f77bcf86cd799439011",
      "planId": "507f1f77bcf86cd799439018",
      "billingCycle": "monthly",
      "orgId": "507f1f77bcf86cd799439013"
    }
  }
}

// Backend automatically:
// 1. Verifies webhook signature
// 2. Verifies payment
// 3. Creates subscription
// 4. Sends confirmation email
// 5. Triggers project webhook
```

---

## Database Operations

### 1. Create Collection Schema

```javascript
// Define collection schema
POST /api/projects/507f1f77bcf86cd799439011/schemas
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "products",
  "fields": [
    {
      "name": "name",
      "type": "string",
      "required": true,
      "indexed": true
    },
    {
      "name": "price",
      "type": "number",
      "required": true
    },
    {
      "name": "description",
      "type": "string"
    },
    {
      "name": "category",
      "type": "string",
      "indexed": true
    },
    {
      "name": "inStock",
      "type": "boolean",
      "default": true
    },
    {
      "name": "createdAt",
      "type": "date",
      "default": "now"
    }
  ],
  "indexes": [
    {
      "fields": ["category", "price"],
      "unique": false
    }
  ]
}

// Response
{
  "success": true,
  "collection": {
    "_id": "507f1f77bcf86cd799439020",
    "name": "products",
    "slug": "products",
    "project": "507f1f77bcf86cd799439011",
    "fields": [
      {
        "name": "name",
        "type": "string",
        "required": true,
        "indexed": true
      },
      {
        "name": "price",
        "type": "number",
        "required": true
      },
      {
        "name": "description",
        "type": "string"
      },
      {
        "name": "category",
        "type": "string",
        "indexed": true
      },
      {
        "name": "inStock",
        "type": "boolean",
        "default": true
      }
    ],
    "createdAt": "2024-01-15T10:50:00Z"
  }
}
```

### 2. Create Document

```javascript
// Create a document in collection
POST /api/projects/507f1f77bcf86cd799439011/data/products
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Laptop",
  "price": 999.99,
  "description": "High-performance laptop",
  "category": "electronics",
  "inStock": true
}

// Response
{
  "message": "Data created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439021",
    "name": "Laptop",
    "price": 999.99,
    "description": "High-performance laptop",
    "category": "electronics",
    "inStock": true,
    "createdAt": "2024-01-15T10:55:00Z",
    "updatedAt": "2024-01-15T10:55:00Z"
  }
}
```

### 3. Query Documents

```javascript
// Get all documents with filters
GET /api/projects/507f1f77bcf86cd799439011/data/products?category=electronics&price[gte]=500&limit=10&page=1
Authorization: Bearer {user_token}

// Response
{
  "data": [
    {
      "_id": "507f1f77bcf86cd799439021",
      "name": "Laptop",
      "price": 999.99,
      "description": "High-performance laptop",
      "category": "electronics",
      "inStock": true,
      "createdAt": "2024-01-15T10:55:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 1,
    "totalPages": 1
  }
}

// Advanced query
POST /api/projects/507f1f77bcf86cd799439011/data/products/query
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "filter": {
    "$and": [
      { "category": "electronics" },
      { "price": { "$gte": 500 } },
      { "inStock": true }
    ]
  },
  "sort": { "price": -1 },
  "limit": 10,
  "skip": 0
}
```

### 4. Update Document

```javascript
// Update document
PATCH /api/projects/507f1f77bcf86cd799439011/data/products/507f1f77bcf86cd799439021
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "price": 899.99,
  "inStock": false
}

// Response
{
  "message": "Data updated successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439021",
    "name": "Laptop",
    "price": 899.99,
    "description": "High-performance laptop",
    "category": "electronics",
    "inStock": false,
    "createdAt": "2024-01-15T10:55:00Z",
    "updatedAt": "2024-01-15T11:00:00Z"
  }
}
```

### 5. Delete Document

```javascript
// Delete document
DELETE /api/projects/507f1f77bcf86cd799439011/data/products/507f1f77bcf86cd799439021
Authorization: Bearer {user_token}

// Response
{
  "message": "Data deleted successfully"
}
```

---

## Real-time Chat System

### 1. Create Chat

```javascript
// Create a new chat
POST /api/projects/507f1f77bcf86cd799439011/chats
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Team Discussion",
  "type": "group", // or "direct"
  "participants": [
    "507f1f77bcf86cd799439012",
    "507f1f77bcf86cd799439022"
  ],
  "metadata": {
    "projectId": "507f1f77bcf86cd799439011"
  }
}

// Response
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439023",
    "name": "Team Discussion",
    "type": "group",
    "participants": [
      {
        "user": "507f1f77bcf86cd799439012",
        "role": "admin",
        "joinedAt": "2024-01-15T11:05:00Z"
      },
      {
        "user": "507f1f77bcf86cd799439022",
        "role": "member",
        "joinedAt": "2024-01-15T11:05:00Z"
      }
    ],
    "createdBy": "507f1f77bcf86cd799439012",
    "project": "507f1f77bcf86cd799439011",
    "createdAt": "2024-01-15T11:05:00Z"
  }
}
```

### 2. Send Message (HTTP)

```javascript
// Send message via HTTP
POST /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "content": "Hello team! How's everyone doing?",
  "type": "text" // or "image", "file", etc.
}

// Response
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439024",
    "content": "Hello team! How's everyone doing?",
    "type": "text",
    "sender": "507f1f77bcf86cd799439012",
    "chat": "507f1f77bcf86cd799439023",
    "project": "507f1f77bcf86cd799439011",
    "createdAt": "2024-01-15T11:06:00Z",
    "updatedAt": "2024-01-15T11:06:00Z"
  }
}
```

### 3. Real-time Messaging (Socket.IO)

```javascript
// Client-side Socket.IO connection
import io from 'socket.io-client';

const socket = io('https://api.mudbase.com', {
  auth: {
    token: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
  }
});

// Join chat room
socket.emit('chat:join', {
  chatId: '507f1f77bcf86cd799439023',
  projectId: '507f1f77bcf86cd799439011'
});

// Send message via Socket.IO
socket.emit('chat:message:send', {
  chatId: '507f1f77bcf86cd799439023',
  content: 'Hello from real-time!',
  type: 'text'
});

// Listen for new messages
socket.on('chat:message:new', (data) => {
  console.log('New message:', data);
  // {
  //   message: {
  //     _id: '507f1f77bcf86cd799439025',
  //     content: 'Hello from real-time!',
  //     sender: { ... },
  //     createdAt: '2024-01-15T11:07:00Z'
  //   },
  //   chatId: '507f1f77bcf86cd799439023'
  // }
});

// Typing indicator
socket.emit('chat:typing', {
  chatId: '507f1f77bcf86cd799439023',
  isTyping: true
});

socket.on('chat:typing', (data) => {
  console.log('User typing:', data);
  // {
  //   userId: '507f1f77bcf86cd799439012',
  //   chatId: '507f1f77bcf86cd799439023',
  //   isTyping: true
  // }
});

// Voice/Video call events
socket.emit('chat:call:initiate', {
  chatId: '507f1f77bcf86cd799439023',
  type: 'video' // or 'voice'
});

socket.on('chat:call:incoming', (data) => {
  console.log('Incoming call:', data);
});

socket.emit('chat:call:accept', {
  callId: 'call_abc123'
});

socket.emit('chat:call:reject', {
  callId: 'call_abc123'
});

socket.emit('chat:call:end', {
  callId: 'call_abc123'
});
```

### 4. Edit Message

```javascript
// Edit message
PATCH /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "content": "Hello team! How's everyone doing? (edited)"
}

// Socket.IO event also emitted: 'chat:message:updated'
```

### 5. Delete Message

```javascript
// Delete message
DELETE /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024
Authorization: Bearer {user_token}

// Socket.IO event also emitted: 'chat:message:deleted'
```

### 6. Add Reaction

```javascript
// Add reaction to message
POST /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024/reactions
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "emoji": "👍"
}

// Socket.IO event also emitted: 'chat:message:reaction'
```

---

## Integration System

### 1. Create Integration

```javascript
// Create integration
POST /api/projects/507f1f77bcf86cd799439011/integrations
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Twitter API Integration",
  "provider": "twitter",
  "category": "social",
    "settings": {
      "isActive": true
      // NOTE: API keys, secrets, and tokens are encrypted and never returned in responses
    },
  "config": {
    "rateLimit": 100,
    "timeout": 5000
  }
}

// Response
{
  "integration": {
    "_id": "507f1f77bcf86cd799439026",
    "name": "Twitter API Integration",
    "provider": "twitter",
    "category": "social",
    "project": "507f1f77bcf86cd799439011",
    "settings": {
      "isActive": true
    },
    "createdAt": "2024-01-15T11:10:00Z",
    "updatedAt": "2024-01-15T11:10:00Z"
  }
}
```

### 2. Execute Integration

```javascript
// Execute integration
POST /api/projects/507f1f77bcf86cd799439011/integrations/507f1f77bcf86cd799439026/execute
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "method": "POST",
  "endpoint": "/2/tweets",
  "body": {
    "text": "Hello from MUDBASE!"
  },
  "headers": {
    "Content-Type": "application/json"
  }
}

// Response
{
  "success": true,
  "data": {
    "status": 201,
    "data": {
      "id": "1234567890",
      "text": "Hello from MUDBASE!",
      "created_at": "2024-01-15T11:12:00Z"
    },
    "headers": {
      "content-type": "application/json",
      "x-rate-limit-remaining": "299"
    }
  },
  "usage": {
    "apiCalls": 1,
    "timestamp": "2024-01-15T11:12:00Z"
  }
}
```

### 3. Get Integration Usage Stats

```javascript
// Get usage statistics
GET /api/projects/507f1f77bcf86cd799439011/integrations/507f1f77bcf86cd799439026/usage?period=month
Authorization: Bearer {user_token}

// Response
{
  "stats": {
    "totalCalls": 1250,
    "successfulCalls": 1200,
    "failedCalls": 50,
    "successRate": 96.0,
    "period": "month",
    "breakdown": [
      {
        "date": "2024-01-15",
        "calls": 45,
        "successful": 43,
        "failed": 2
      }
    ]
  }
}
```

---

## Complete End-to-End Scenarios

### Scenario 1: E-commerce Platform Setup

```javascript
// Step 1: Register Admin User
POST /api/auth/local/register
{
  "email": "admin@ecommerce.com",
  "password": "SecurePass123!",
  "firstName": "Admin",
  "lastName": "User"
}

// Step 2: Create Organization
POST /api/orgs
Authorization: Bearer {admin_token}
{
  "name": "Ecommerce Corp"
}

// Step 3: Create Project
POST /api/projects
Authorization: Bearer {admin_token}
{
  "name": "Ecommerce Platform",
  "orgId": "{org_id}"
}

// Step 4: Setup Payment Gateway
POST /api/billing/orgs/{org_id}/payment-gateway
Authorization: Bearer {admin_token}
{
  "provider": "paystack",
  "publicKey": "pk_test_...",
  "secretKey": "sk_test_..."
}

// Step 5: Create Product Schema
POST /api/projects/{project_id}/schemas
Authorization: Bearer {admin_token}
{
  "name": "products",
  "fields": [
    { "name": "name", "type": "string", "required": true },
    { "name": "price", "type": "number", "required": true },
    { "name": "stock", "type": "number", "required": true }
  ]
}

// Step 6: Create Order Schema
POST /api/projects/{project_id}/schemas
Authorization: Bearer {admin_token}
{
  "name": "orders",
  "fields": [
    { "name": "customerId", "type": "string", "required": true },
    { "name": "products", "type": "array", "required": true },
    { "name": "total", "type": "number", "required": true },
    { "name": "status", "type": "string", "default": "pending" }
  ]
}

// Step 7: Customer Registration
POST /api/auth/local/register
{
  "email": "customer@example.com",
  "password": "CustomerPass123!",
  "firstName": "John",
  "lastName": "Customer",
  "projectId": "{project_id}"
}

// Step 8: Customer Creates Order
POST /api/projects/{project_id}/data/orders
Authorization: Bearer {customer_token}
{
  "customerId": "{customer_id}",
  "products": [
    { "productId": "{product_id}", "quantity": 2 }
  ],
  "total": 1999.98,
  "status": "pending"
}

// Step 9: Process Payment
POST /api/billing/public/projects/{project_id}/checkout
{
  "planId": "{plan_id}",
  "billingCycle": "one-time",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  },
  "amount": 1999.98
}

// Step 10: Update Order Status
PATCH /api/projects/{project_id}/data/orders/{order_id}
Authorization: Bearer {admin_token}
{
  "status": "paid"
}
```

### Scenario 2: Crypto Wallet Integration

```javascript
// Step 1: User generates Bitcoin wallet key
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
{
  "currency": "BTC"
}

// Response includes privateKey (shown once only) that user must save securely

// Step 2: User creates wallet with generated key
POST /api/wallet/create
Authorization: Bearer {user_token}
{
  "currency": "BTC",
  "privateKey": "[PRIVATE_KEY_FROM_STEP_1]",
  "address": "[ADDRESS_FROM_STEP_1]"
}

// Response does NOT include privateKey - it's encrypted and stored securely

// Step 3: Check wallet balance
GET /api/wallet/{wallet_id}/balance
Authorization: Bearer {user_token}

// Step 4: Receive payment (external)
// Bitcoin sent to wallet address

// Step 5: Withdraw funds (async processing)
POST /api/wallet/{wallet_id}/withdraw
Authorization: Bearer {user_token}
{
  "toAddress": "bc1q...",
  "amount": 0.01,
  "feeRate": 10
}

// Response (returns immediately - async processing)
{
  "success": true,
  "data": {
    "transactionId": "...",
    "status": "queued",
    "platformFee": 0.0005 // max(1% * 0.01, 0.0005 BTC minimum)
  }
}

// Step 6: Check transaction status (poll until confirmed)
GET /api/wallet/transactions/{transaction_id}
Authorization: Bearer {user_token}

// Response (after confirmation - may take 10-60 minutes for BTC)
{
  "success": true,
  "data": {
    "status": "confirmed",
    "mainTxHash": "...",
    "mainTxStatus": "confirmed"
  }
}
```

### Scenario 3: SaaS Subscription Flow

```javascript
// Step 1: Setup billing plan
POST /api/billing/projects/{project_id}/plans
Authorization: Bearer {admin_token}
{
  "name": "Premium",
  "pricing": {
    "monthly": { "amount": 10000, "currency": "NGN" },
    "yearly": { "amount": 100000, "currency": "NGN" }
  },
  "features": [
    { "name": "api_access", "included": true },
    { "name": "storage", "included": true, "limit": 107374182400 }
  ]
}

// Step 2: Customer subscribes
POST /api/billing/public/projects/{project_id}/checkout
{
  "planId": "{plan_id}",
  "billingCycle": "monthly",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  }
}

// Step 3: Check subscription status
GET /api/billing/public/projects/{project_id}/subscription?email=customer@example.com

// Step 4: Check feature access
GET /api/billing/public/projects/{project_id}/feature-access?email=customer@example.com&feature=api_access

// Step 5: Record usage
POST /api/billing/public/projects/{project_id}/usage
{
  "email": "customer@example.com",
  "metric": "api_calls",
  "quantity": 150
}
```

---

## API Reference Quick Guide

### Authentication Endpoints

```
POST   /api/auth/register                - Register new user (org-based)
POST   /api/auth/login                   - Login user (org-based)
POST   /api/auth/logout                  - Logout user (org-based)
GET    /api/auth/session                 - Get current session (org-based)
POST   /api/auth/password-reset          - Request password reset (org-based)
POST   /api/auth/password-reset/{token}  - Reset password (org-based)
POST   /api/auth/local/register          - Register new user (project-based)
POST   /api/auth/local/login             - Login user (project-based)
POST   /api/auth/local/logout            - Logout user (project-based)
GET    /api/auth/local/session           - Get current session (project-based)
POST   /api/auth/local/password-reset    - Request password reset (project-based)
POST   /api/auth/local/password-reset/{token} - Reset password (project-based)
GET    /api/auth/oauth/{provider}        - Initiate OAuth
GET    /api/auth/oauth/{provider}/callback - OAuth callback
POST   /api/auth/magic-link/send         - Send magic link
GET    /api/auth/magic-link/verify        - Verify magic link
POST   /api/auth/otp/send                - Send OTP
POST   /api/auth/otp/verify              - Verify OTP
POST   /api/auth/2fa/setup               - Setup 2FA
POST   /api/auth/2fa/verify              - Verify 2FA
```

### Wallet Endpoints

```
GET    /api/wallet/currencies            - Get supported currencies
POST   /api/wallet/generate-key          - Generate key pair
POST   /api/wallet/validate-address      - Validate address
POST   /api/wallet/create                 - Create wallet
GET    /api/wallet                        - Get user wallets
GET    /api/wallet/{id}/balance          - Get wallet balance
GET    /api/wallet/{id}/private-key      - Get wallet private key (WARNING: Sensitive)
POST   /api/wallet/{id}/withdraw          - Withdraw funds
```

### Billing Endpoints

```
POST   /api/billing/orgs/{orgId}/payment-gateway - Create gateway
GET    /api/billing/orgs/{orgId}/payment-gateway - Get gateway status
POST   /api/billing/projects/{id}/plans         - Create plan
GET    /api/billing/projects/{id}/plans         - Get plans
GET    /api/billing/public/projects/{id}/plans  - Get public plans
POST   /api/billing/public/projects/{id}/checkout - Create checkout
POST   /api/billing/public/projects/{id}/verify-payment - Verify payment
GET    /api/billing/public/projects/{id}/subscription - Check subscription
GET    /api/billing/public/projects/{id}/feature-access - Check feature access
POST   /api/billing/public/projects/{id}/usage   - Record usage
POST   /api/billing/subscriptions/{id}/cancel   - Cancel subscription
POST   /api/billing/webhooks/{provider}         - Payment webhook
```

### Database Endpoints

```
POST   /api/projects/{id}/schemas         - Create schema
GET    /api/projects/{id}/schemas         - Get schemas
POST   /api/projects/{id}/data/{collection} - Create document
GET    /api/projects/{id}/data/{collection} - Query documents
PATCH  /api/projects/{id}/data/{collection}/{id} - Update document
DELETE /api/projects/{id}/data/{collection}/{id} - Delete document
POST   /api/projects/{id}/data/{collection}/query - Advanced query
```

### Chat Endpoints

```
POST   /api/projects/{id}/chats            - Create chat
GET    /api/projects/{id}/chats           - Get user chats
POST   /api/projects/{id}/chats/{id}/messages - Send message
GET    /api/projects/{id}/chats/{id}/messages - Get messages
PATCH  /api/projects/{id}/chats/{id}/messages/{id} - Edit message
DELETE /api/projects/{id}/chats/{id}/messages/{id} - Delete message
POST   /api/projects/{id}/chats/{id}/messages/{id}/reactions - Add reaction
```

### Integration Endpoints

```
GET    /api/projects/{id}/integrations/templates - Get templates
POST   /api/projects/{id}/integrations          - Create integration
GET    /api/projects/{id}/integrations          - Get integrations
POST   /api/projects/{id}/integrations/{id}/test - Test integration
POST   /api/projects/{id}/integrations/{id}/execute - Execute integration
GET    /api/projects/{id}/integrations/{id}/usage - Get usage stats
```

---

## Security Features

### 1. Session Management
- **Idle Timeout**: 30 minutes of inactivity
- **Rolling Sessions**: Session extends on activity
- **Secure Cookies**: HttpOnly, Secure in production

### 2. Access Control
- **RBAC**: Role-Based Access Control (owner, admin, developer, viewer)
- **Project-Level Access**: Users can only access their project's resources
- **Organization-Level Access**: Users can only access their org's resources

### 3. Encryption
- **Private Keys**: AES-256-GCM encryption at rest
- **API Credentials**: Encrypted in database
- **Password Hashing**: bcrypt with salt rounds 12

### 4. Rate Limiting
- **Global**: 100 requests per 15 minutes per IP
- **Authentication**: 5 login attempts per 15 minutes
- **API Keys**: Configurable per project

### 5. Audit Logging
- All authentication events logged
- All authorization failures logged
- All sensitive operations logged

---

## Best Practices

### 1. Authentication
- Always use HTTPS in production
- Store JWT tokens securely (httpOnly cookies recommended)
- Implement token refresh mechanism
- Use 2FA for sensitive accounts

### 2. Wallet Management
- Never log private keys
- Always validate addresses before transactions
- Use custom keys only when necessary
- Keep backup of private keys securely
- **Async Processing**: Withdrawals are processed asynchronously - check transaction status for updates
- **Real Confirmations**: System uses real blockchain confirmations (not fake delays)
- **Minimum Fees**: Enforced per currency to ensure profitability
- **Status Tracking**: Monitor transaction status through: `queued` → `prefunding` → `processing` → `broadcasted` → `confirmed` → `completed`

### 3. Billing
- Verify webhook signatures
- Handle payment failures gracefully
- Implement retry logic for failed payments
- Monitor subscription status

### 4. Database
- Use indexes for frequently queried fields
- Implement pagination for large datasets
- Validate input data
- Use transactions for critical operations

### 5. Real-time
- Handle connection failures
- Implement reconnection logic
- Use rooms for efficient message delivery
- Clean up on disconnect

---

## Error Handling

### Common Error Codes

```javascript
// Authentication Errors
401 - Unauthorized (Invalid token)
403 - Forbidden (Insufficient permissions)
429 - Too Many Requests (Rate limit exceeded)

// Validation Errors
400 - Bad Request (Invalid input)
422 - Unprocessable Entity (Validation failed)

// Resource Errors
404 - Not Found (Resource doesn't exist)
409 - Conflict (Resource already exists)

// Server Errors
500 - Internal Server Error
503 - Service Unavailable
```

### Error Response Format

```javascript
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "email": "Email is required",
      "password": "Password must be at least 8 characters"
    }
  }
}
```

---

## Project Fee Management

### Overview

Project owners can configure their own fees per currency, which are collected in addition to platform fees. These fees are accumulated and paid out via automated bi-weekly payouts.

### 1. Configure Project Fee Settings

```javascript
// Step 1: Create or update fee settings
POST /api/projects/507f1f77bcf86cd799439011/fee-settings
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "enabled": true,
  "feeAmount": 0.00005, // Must be < platform minimum fee (0.00012 BTC)
  "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "payoutThreshold": 0.001 // Minimum amount before payout
}

// Response
{
  "success": true,
  "message": "Fee settings updated successfully",
  "data": {
    "feeSettings": {
      "BTC": {
        "enabled": true,
        "feeAmount": 0.00005,
        "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "payoutThreshold": 0.001,
        "addressVerified": false
      }
    }
  }
}
```

### 2. Verify Payout Address

```javascript
// Step 1: Initiate address verification
POST /api/projects/507f1f77bcf86cd799439011/fee-settings/BTC/verify-address
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "message": "Test transaction sent. Please confirm receipt or provide transaction hash from the address.",
  "data": {
    "verificationStatus": "pending",
    "testTxHash": "abc123def456...",
    "testAmount": 0.00001,
    "instructions": "Either confirm you received the test transaction, or send a transaction FROM the payout address to prove ownership."
  }
}

// Step 2: Confirm verification
POST /api/projects/507f1f77bcf86cd799439011/fee-settings/BTC/confirm-verification
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "txHash": "xyz789abc123..." // Transaction hash proving ownership
}

// Response
{
  "success": true,
  "message": "Address verified successfully",
  "data": {
    "verified": true,
    "verifiedAt": "2024-01-15T12:00:00Z"
  }
}
```

### 3. Check Fee Balance

```javascript
// Get balance for specific currency
GET /api/projects/507f1f77bcf86cd799439011/fee-balances/BTC
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "currency": "BTC",
    "collectedAmount": 0.0005,
    "threshold": 0.001,
    "status": "accumulating",
    "nextScheduledPayoutDate": null,
    "lastPayoutDate": null,
    "totalPaidOut": 0,
    "totalCollected": 0.0005
  }
}

// Get all balances
GET /api/projects/507f1f77bcf86cd799439011/fee-balances
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "balances": [
      {
        "currency": "BTC",
        "collectedAmount": 0.0005,
        "threshold": 0.001,
        "status": "accumulating"
      },
      {
        "currency": "ETH",
        "collectedAmount": 0.002,
        "threshold": 0.001,
        "status": "ready",
        "nextScheduledPayoutDate": "2024-01-17T02:00:00Z"
      }
    ]
  }
}
```

### 4. View Payout History

```javascript
// Get payout history
GET /api/projects/507f1f77bcf86cd799439011/payout-history?limit=10&offset=0
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "payouts": [
      {
        "_id": "507f1f77bcf86cd799439030",
        "currency": "BTC",
        "grossAmount": 0.001,
        "networkFee": 0.0001,
        "netAmount": 0.0009,
        "toAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "txHash": "abc123def456...",
        "status": "completed",
        "scheduledAt": "2024-01-15T02:00:00Z",
        "processedAt": "2024-01-15T02:05:00Z",
        "confirmedAt": "2024-01-15T02:45:00Z",
        "createdAt": "2024-01-15T02:00:00Z"
      }
    ],
    "total": 1,
    "limit": 10,
    "offset": 0
  }
}
```

### 5. Request Manual Payout

```javascript
// Request manual payout (restricted: once per 30 days per currency)
POST /api/projects/507f1f77bcf86cd799439011/payouts/request-manual
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC"
}

// Response
{
  "success": true,
  "message": "Manual payout requested and processed",
  "data": {
    "payout": {
      "_id": "507f1f77bcf86cd799439031",
      "currency": "BTC",
      "netAmount": 0.0009,
      "txHash": "abc123def456...",
      "status": "completed"
    }
  }
}
```

### 6. Fee Dashboard

```javascript
// Get comprehensive fee dashboard
GET /api/projects/507f1f77bcf86cd799439011/fee-dashboard
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "feeSettings": {
      "BTC": {
        "enabled": true,
        "feeAmount": 0.00005,
        "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "addressVerified": true
      }
    },
    "balances": [
      {
        "currency": "BTC",
        "collectedAmount": 0.0005,
        "threshold": 0.001,
        "status": "accumulating",
        "totalPaidOut": 0.0009,
        "totalCollected": 0.0014
      }
    ],
    "recentPayouts": [
      {
        "_id": "507f1f77bcf86cd799439030",
        "currency": "BTC",
        "netAmount": 0.0009,
        "status": "completed",
        "createdAt": "2024-01-15T02:00:00Z"
      }
    ],
    "totalEarned": 0.0009
  }
}
```

### Important Notes

1. **Fee Validation**: Project fee must be less than platform minimum fee (cannot undercut platform)
2. **Address Verification**: Required before fees are collected (prevents fraud)
3. **Automated Payouts**: Bi-weekly schedule (Wed/Sat 2 AM UTC) when threshold is met
4. **Network Fees**: Deducted from payout amount (platform pays network fees)
5. **Manual Payouts**: Restricted to once per 30 days per currency, requires 25% of threshold
6. **Fee Collection**: Automatically collected during user withdrawals if enabled and verified

---

## Conclusion

This document provides a comprehensive overview of the MUDBASE Backend-as-a-Service platform, including:

- **Multiple Authentication Methods**: Local, OAuth, Magic Link, OTP, 2FA
- **Wallet as a Service**: Support for 7 cryptocurrencies with custom key support
- **Project Fee System**: Project owners can set their own fees with automated bi-weekly payouts
- **Billing System**: Nigerian payment gateways (Paystack, Flutterwave) with subscription management
- **Real-time Features**: Socket.IO-based chat system
- **Database Operations**: Flexible schema-based collections
- **Integration System**: 50+ API integrations
- **Security**: RBAC, encryption, rate limiting, audit logging

For more detailed API documentation, refer to the OpenAPI specification at `/api-docs`.


## Table of Contents
1. [System Architecture Overview](#system-architecture-overview)
2. [Authentication Flows](#authentication-flows)
3. [Organization & Project Setup](#organization--project-setup)
4. [Wallet as a Service Workflows](#wallet-as-a-service-workflows)
5. [Billing & Payment Gateway Workflows](#billing--payment-gateway-workflows)
6. [Database Operations](#database-operations)
7. [Real-time Chat System](#real-time-chat-system)
8. [Integration System](#integration-system)
9. [Complete End-to-End Scenarios](#complete-end-to-end-scenarios)
10. [API Reference Quick Guide](#api-reference-quick-guide)

---

## System Architecture Overview

### Core Components

```
┌────────────────────────────────────────────────────────────┐
│                    MUDBASE Backend Platform                │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Auth Layer  │  │  API Layer   │  │  Socket.IO   │      │
│  │              │  │              │  │  Real-time   │      │
│  │ • Local      │  │ • REST API   │  │ • Chat       │      │
│  │ • OAuth      │  │ • Webhooks   │  │ • Events     │      │
│  │ • Magic Link │  │ • GraphQL    │  │ • Database   │      │
│  │ • OTP        │  │              │  │              │      │
│  │ • 2FA        │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Database   │  │   Storage    │  │  Integrations│      │
│  │              │  │              │  │              │      │
│  │ • MongoDB    │  │ • File Store │  │ • 50+ APIs   │      │
│  │ • Collections│  │ • Buckets    │  │ • Webhooks   │      │
│  │ • Indexes    │  │ • CDN        │  │ • Custom     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Wallet     │  │   Billing    │  │  Security    │      │
│  │   Service    │  │              │  │              │      │
│  │              │  │ • Paystack   │  │ • RBAC       │      │
│  │ • BTC        │  │ • Flutterwave│  │ • Encryption │      │
│  │ • ETH        │  │ • Plans      │  │ • Rate Limit│       │
│  │ • SOL        │  │ • Usage      │  │ • Audit Log  │      │
│  │ • TRX        │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└────────────────────────────────────────────────────────────┘
```

### Data Flow

```
User Request → Authentication → Authorization (RBAC) → Business Logic → Database/External API → Response
                ↓
            JWT Token
                ↓
        Session Management (30min idle timeout)
                ↓
        Real-time Updates via Socket.IO
```

---

## Authentication Flows

### 1. Basic Authentication (Organization-based)

These endpoints are for organization-level authentication and create organizations automatically.

#### Registration Flow

```javascript
// Step 1: User Registration (creates organization automatically)
POST /api/auth/register
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "firstName": "John",
  "lastName": "Doe",
  "orgName": "Acme Corp" // Optional
}

// Response
{
  "message": "User registered successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner",
    "emailVerified": false,
    "org": "507f1f77bcf86cd799439013"
  }
}

// Step 2: Login
POST /api/auth/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]"
}

// Response
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  }
}

// Step 3: Get Session
GET /api/auth/session
Authorization: Bearer {token}

// Response
{
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner"
  },
  "authenticated": true
}

// Step 4: Password Reset Request
POST /api/auth/password-reset
Content-Type: application/json

{
  "email": "john.doe@example.com"
}

// Response
{
  "message": "Password reset email sent"
}

// Step 5: Reset Password
POST /api/auth/password-reset/{token}
Content-Type: application/json

{
  "password": "[NEW_PASSWORD]"
}

// Response
{
  "message": "Password reset successful"
}
```

### 2. Local Authentication (Project-based)

These endpoints are for project-level authentication and require a projectId.

#### Registration Flow

```javascript
// Step 1: User Registration
POST /api/auth/local/register
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "firstName": "John",
  "lastName": "Doe",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "message": "User registered successfully",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "emailVerified": false,
    "role": "developer",
    "org": "507f1f77bcf86cd799439013"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

// Step 2: Email Verification (Optional)
POST /api/auth/verify-email
Authorization: Bearer {token}
Content-Type: application/json

{
  "token": "email-verification-token-from-email"
}

// Response
{
  "success": true,
  "message": "Email verified successfully"
}

// Step 3: Login
POST /api/auth/local/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  },
  "expiresIn": "24h"
}
```

#### Password Reset (Project-based)

```javascript
// Step 1: Request password reset
POST /api/auth/local/password-reset
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "projectId": "507f1f77bcf86cd799439011" // Optional
}

// Response
{
  "message": "Password reset email sent"
}

// Step 2: Reset password
POST /api/auth/local/password-reset/{token}
Content-Type: application/json

{
  "password": "[NEW_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011" // Optional
}

// Response
{
  "message": "Password reset successful"
}
```

#### Session Management

```javascript
// Session expires after 30 minutes of inactivity
// Server automatically extends session on activity (rolling: true)

// Check current session (organization-based)
GET /api/auth/session
Authorization: Bearer {token}

// Check current session (project-based)
GET /api/auth/local/session?projectId={projectId}
Authorization: Bearer {token}

// Response
{
  "user": {
    "id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "project": {
      "id": "507f1f77bcf86cd799439011",
      "name": "My Awesome App",
      "role": "developer"
    }
  },
  "authenticated": true
}

// Response
{
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  }
}

// Refresh token (if implemented)
POST /api/auth/refresh
Authorization: Bearer {token}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": "24h"
}

// Logout (organization-based)
POST /api/auth/logout
Authorization: Bearer {token}

// Response
{
  "message": "Logout successful"
}

// Logout (project-based)
POST /api/auth/local/logout
Authorization: Bearer {token}

// Response
{
  "message": "Logout successful"
}
```

**Key Differences Between Organization-based and Project-based Auth:**

- **Organization-based** (`/api/auth/*`): Creates organizations automatically, simpler flow, good for admin/owner accounts
- **Project-based** (`/api/auth/local/*`): Requires existing project, project-specific authentication, includes rate limiting and captcha verification, better for end-user authentication

### 3. OAuth Authentication

#### Supported Providers
- Google, GitHub, Facebook, Microsoft, Apple, LinkedIn, Discord, Twitter, etc.

#### OAuth Flow Example (Google)

```javascript
// Step 1: Initiate OAuth
// Redirect user to:
GET /api/auth/oauth/google?projectId=507f1f77bcf86cd799439011

// User is redirected to Google consent screen
// After consent, Google redirects to:
GET /api/auth/oauth/google/callback?code={authorization_code}

// Step 2: Backend processes OAuth callback
// Backend automatically:
// 1. Exchanges code for access token
// 2. Fetches user profile from Google
// 3. Creates/updates user in database
// 4. Generates JWT token
// 5. Redirects to frontend with token

// Frontend receives redirect:
https://yourapp.com/auth/callback?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// Step 3: Use token for authenticated requests
Authorization: Bearer {token}

// Response (from OAuth callback redirect)
// Frontend receives:
https://yourapp.com/auth/callback?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...&user={"id":"507f1f77bcf86cd799439012","email":"john.doe@example.com"}
```

#### OAuth Provider Configuration

```javascript
// Configure OAuth provider in project settings
PATCH /api/projects/{projectId}
Authorization: Bearer {admin_token}
Content-Type: application/json

{
  "authProviders": {
    "google": {
      "enabled": true,
      "clientId": "your-google-client-id",
      "clientSecret": "your-google-client-secret",
      "callbackUrl": "https://api.mudbase.com/api/auth/oauth/google/callback"
    },
    "github": {
      "enabled": true,
      "clientId": "your-github-client-id",
      "clientSecret": "your-github-client-secret"
    }
  }
}
```

### 4. Magic Link Authentication

```javascript
// Step 1: Request Magic Link
POST /api/auth/magic-link/send
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "projectId": "507f1f77bcf86cd799439011",
  "redirectUrl": "https://yourapp.com/auth/verify"
}

// Response
{
  "success": true,
  "message": "Magic link sent to email"
}

// Step 2: User clicks link in email
// Email contains: https://api.mudbase.com/api/auth/magic-link/verify?token={magic_token}

// Step 3: Verify Magic Link
GET /api/auth/magic-link/verify?token={magic_token}&projectId={projectId}

// Response (redirects to frontend with token)
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

### 5. OTP Authentication (SMS/Email)

```javascript
// Step 1: Request OTP
POST /api/auth/otp/send
Content-Type: application/json

{
  "identifier": "john.doe@example.com", // or phone number
  "method": "email", // or "sms" or "auto"
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "message": "OTP sent to email",
  "expiresIn": 300 // 5 minutes
}

// Step 2: Verify OTP
POST /api/auth/otp/verify
Content-Type: application/json

{
  "identifier": "john.doe@example.com",
  "otp": "123456",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

### 6. Two-Factor Authentication (2FA)

```javascript
// Step 1: Enable 2FA
POST /api/auth/2fa/setup
Authorization: Bearer {token}
Content-Type: application/json

// Response
{
  "secret": "JBSWY3DPEHPK3PXP",
  "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..."
}

// Step 2: Verify and Enable
POST /api/auth/2fa/verify
Authorization: Bearer {token}
Content-Type: application/json

{
  "token": "123456" // TOTP code from authenticator app
}

// Step 3: Login with 2FA
POST /api/auth/local/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response (requires 2FA)
{
  "requires2FA": true,
  "tempToken": "temp-token-for-2fa-verification"
}

// Step 4: Verify 2FA
POST /api/auth/2fa/verify-login
Content-Type: application/json

{
  "tempToken": "temp-token-for-2fa-verification",
  "token": "123456"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

---

## Organization & Project Setup

### Complete Setup Workflow

```javascript
// Step 1: Create Organization
POST /api/orgs
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Acme Corporation",
  "slug": "acme-corp" // auto-generated if not provided
}

// Response
{
  "success": true,
  "org": {
    "_id": "507f1f77bcf86cd799439013",
    "name": "Acme Corporation",
    "slug": "acme-corp",
    "members": ["507f1f77bcf86cd799439012"],
    "createdAt": "2024-01-15T10:00:00Z"
  }
}

// Step 2: Create Project
POST /api/projects
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "My Awesome App",
  "orgId": "507f1f77bcf86cd799439013",
  "description": "A revolutionary app"
}

// Response
{
  "success": true,
  "project": {
    "_id": "507f1f77bcf86cd799439011",
    "name": "My Awesome App",
    "slug": "my-awesome-app",
    "org": "507f1f77bcf86cd799439013",
    "apiKey": "pk_live_abc123...",
    // NOTE: secretKey is NEVER returned in responses
    "settings": {
      "auth": {
        "requireEmailVerification": true,
        "sessionTimeout": 1800000
      }
    }
  }
}

// Step 3: Configure Project Settings
PATCH /api/projects/507f1f77bcf86cd799439011
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "settings": {
    "auth": {
      "providers": {
        "local": { "enabled": true },
        "google": { "enabled": true },
        "github": { "enabled": true },
        "magicLink": { "enabled": true },
        "otp": { "enabled": true }
      },
      "requireEmailVerification": true,
      "sessionTimeout": 1800000,
      "passwordPolicy": {
        "minLength": 8,
        "requireUppercase": true,
        "requireNumbers": true,
        "requireSpecialChars": true
      }
    },
    "database": {
      "collections": {
        "users": { "enabled": true },
        "products": { "enabled": true },
        "orders": { "enabled": true }
      }
    },
    "storage": {
      "maxFileSize": 10485760, // 10MB
      "allowedTypes": ["image/jpeg", "image/png", "application/pdf"]
    }
  }
}

// Step 4: Set up Payment Gateway (for billing)
POST /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "provider": "paystack",
  "publicKey": "pk_test_abc123...",
  "secretKey": "[SECRET_KEY_NOT_RETURNED]", // Only sent during creation, never returned
  "config": {
    "currency": "NGN",
    "webhookUrl": "https://api.mudbase.com/api/billing/webhooks/paystack"
  }
}

// Response
{
  "message": "Payment gateway created successfully",
  "gateway": {
    "_id": "507f1f77bcf86cd799439014",
    "provider": "paystack",
    "status": "pending"
    // NOTE: secretKey and webhookSecret are NEVER returned in responses
  }
}

// Step 5: Activate Payment Gateway
PATCH /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "status": "active"
}
```

---

## Wallet as a Service Workflows

### Important Security Notes

1. **Private Key Generation**: Each wallet gets its own unique private key. Keys are generated per wallet, not shared per project or organization.
2. **Private Key Visibility**: Private keys are ONLY shown once in the `/api/wallet/generate-key` endpoint response. They are NEVER returned in:
   - Wallet creation responses
   - Wallet listing responses
   - Wallet balance responses
   - Any other wallet-related endpoints
3. **Encryption**: All private keys are encrypted using AES-256-GCM before storage in the database.
4. **Key Storage**: Users must save the private key securely when generated. It cannot be retrieved later.

### Understanding Wallet Encryption

**Two Types of Keys**:

1. **Project Encryption Key** (`walletEncryptionKey`):
   - One per project (auto-generated)
   - Used to encrypt/decrypt wallet private keys for database storage
   - Provides additional security layer and project isolation
   - **NOT used for signing transactions**

2. **Wallet Private Keys**:
   - One unique key per wallet (generated per wallet)
   - The actual cryptocurrency private key
   - Used to sign blockchain transactions
   - Encrypted using project encryption key before storage

**Transaction Flow**:
- When making a transaction, the wallet's private key is decrypted using the project encryption key
- The decrypted wallet private key is then used to sign the transaction
- The project encryption key is NEVER used to sign transactions - only the wallet's private key

For detailed explanation, see `docs/WALLET_ENCRYPTION_EXPLAINED.md`.

### Supported Cryptocurrencies
- **BTC** (Bitcoin) - Bech32 addresses (bc1...)
- **ETH** (Ethereum)
- **BNB** (Binance Smart Chain)
- **SOL** (Solana)
- **TRX** (Tron)
- **LTC** (Litecoin)
- **USDT** (Tether - Multi-network: ETH, TRX, BSC, SOL, POLYGON)

### Platform Fee Structure (Hybrid Model)

**Formula**: `Platform Fee = max(1% * amount, minimumFee)`

| Currency | Minimum Fee | Minimum Withdrawal | Notes |
|----------|-------------|-------------------|-------|
| BTC | 0.00012 BTC (~$8) | 0.0002 BTC (~$13) | Updated based on market rates |
| ETH | 0.001 ETH (~$2.50) | 0.001 ETH (~$2.50) | Increased to protect against gas spikes |
| BNB | 0.0002 BNB (~$0.13) | 0.001 BNB (~$0.65) | Updated based on market rates |
| SOL | 0.008 SOL (~$1.20) | 0.01 SOL (~$1.50) | Platform as feePayer (no pre-fund) |
| TRX | 1 TRX (~$0.05) | 5 TRX (~$0.25) | Lowered for micro-transactions support |
| LTC | 0.0001 LTC (~$0.01) | 0.001 LTC (~$0.10) | Updated based on market rates |
| USDT-ETH | $4 fixed | 5 USDT | Covers gas spikes |
| USDT-BSC | $0.75 fixed | 2 USDT | Scaled with network cost + profit margin |
| USDT-TRX | $0.50 fixed | 2 USDT | Scaled with network cost + profit margin |
| USDT-SOL | $0.15 fixed | 2 USDT | Scaled with network cost + profit margin |

**Profitability Guardrails**: Transactions are rejected if platform fee < estimated network cost + pre-fund cost.

**Important Notes**:
- **ETH Gas Spikes**: ETH minimum fee (0.001 ETH) protects against gas spikes, but network conditions may still cause fees to exceed minimum during extreme congestion. Consider implementing dynamic fee adjustments for ETH during high gas periods.
- **USDT Fees**: Fees on non-ETH chains (BSC, TRX, SOL) are scaled with network cost + profit margin rather than flat rates for better user experience and competitiveness.
- **Micro-Transactions**: TRX minimum withdrawal lowered to 5 TRX to support small users and micro-transactions.

For detailed fee structure and rationale, see [FEE_STRUCTURE_UPDATED.md](./FEE_STRUCTURE_UPDATED.md)

### 1. Generate Private Key (Dashboard Endpoint)

**Important**: Each wallet gets its own unique private key. Keys are generated per wallet, not per project. The private key is only shown once in this response and must be saved securely.

```javascript
// Generate a new key pair for any supported currency
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "network": null // Only required for USDT
}

// Response
{
  "success": true,
  "data": {
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "privateKey": "L5EZftvrYaSudiozVRzTqLcHLNDoVn7H5HSfM9BAN6pMA5zoi3WZ",
    "publicKey": "02a1633cafcc01ebfb6d78e39f687a1f0995c62fc95f51ead10a02ee0be551b5fb"
  },
  "warning": "This private key is shown only once. Store it securely."
}

// For USDT with network
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "USDT",
  "network": "ETH" // Options: ETH, TRX, BSC, SOL, POLYGON
}

// Response
{
  "success": true,
  "data": {
    "currency": "USDT",
    "network": "ETH",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "privateKey": "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
    "publicKey": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
  },
  "warning": "This private key is shown only once. Store it securely."
}
```

### 2. Create Wallet with Generated Private Key

**Security Note**: Private keys are NEVER returned in wallet creation responses. They are encrypted and stored securely. Only the address and wallet metadata are returned.

```javascript
// User can set the generated private key from dashboard
POST /api/wallet/create
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "privateKey": "[PRIVATE_KEY_FROM_GENERATE_KEY_ENDPOINT]",
  "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh"
}

// Response
{
  "success": true,
  "message": "BTC wallet created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439015",
    "user": "507f1f77bcf86cd799439012",
    "org": "507f1f77bcf86cd799439013",
    "project": "507f1f77bcf86cd799439011",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "balance": "0",
    "isCustomKey": true,
    "isActive": true,
    "createdAt": "2024-01-15T10:00:00Z",
    "updatedAt": "2024-01-15T10:00:00Z"
  }
}
```

### 3. Create Wallet with Auto-Generated Key

```javascript
// Let system generate the key pair
POST /api/wallet/create
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "ETH"
}

// Response
{
  "success": true,
  "message": "ETH wallet created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439016",
    "user": "507f1f77bcf86cd799439012",
    "org": "507f1f77bcf86cd799439013",
    "project": "507f1f77bcf86cd799439011",
    "currency": "ETH",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "balance": "0",
    "isCustomKey": false,
    "isActive": true,
    "createdAt": "2024-01-15T10:00:00Z",
    "updatedAt": "2024-01-15T10:00:00Z"
  }
}
```

### 4. Get User Wallets

```javascript
// Get all wallets for authenticated user
GET /api/wallet
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": [
    {
      "_id": "507f1f77bcf86cd799439015",
      "currency": "BTC",
      "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
      "balance": "0.05",
      "isCustomKey": true,
      "isActive": true,
      "project": "507f1f77bcf86cd799439011",
      "createdAt": "2024-01-15T10:00:00Z"
    },
    {
      "_id": "507f1f77bcf86cd799439016",
      "currency": "ETH",
      "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
      "balance": "2.5",
      "isCustomKey": false,
      "isActive": true,
      "project": "507f1f77bcf86cd799439011",
      "createdAt": "2024-01-15T10:00:00Z"
    }
  ]
}
```

### 4.5. Get Wallet Private Key

**Security Warning**: This endpoint returns the decrypted private key. Use only when you need to export your wallet. The private key is sensitive and should be kept secure.

```javascript
// Get private key for a specific wallet
GET /api/wallet/{walletId}/private-key
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "walletId": "507f1f77bcf86cd799439015",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "privateKey": "L5EZftvrYaSudiozVRzTqLcHLNDoVn7H5HSfM9BAN6pMA5zoi3WZ",
    "isCustomKey": true
  },
  "warning": "Keep this private key secure and never share it. Anyone with access to this key can control your wallet.",
  "security": {
    "accessedAt": "2024-01-15T11:20:00Z",
    "accessedBy": "507f1f77bcf86cd799439012"
  }
}
```

**Access Control**:
- User can only retrieve private keys for wallets they own
- Wallet must belong to the user's organization
- All access is logged for security audit

### 6. Get Wallet Balance

```javascript
// Get balance for specific wallet
GET /api/wallet/507f1f77bcf86cd799439015/balance
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "walletId": "507f1f77bcf86cd799439015",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "balance": "0.05",
    "balanceInUSD": 2500.00,
    "lastSyncedAt": "2024-01-15T10:30:00Z"
  }
}
```

### 7. Withdraw Funds (Async Processing)

**Important**: Withdrawals are processed asynchronously. The API returns immediately with a `queued` status. You must check the transaction status to see when it's confirmed. Real blockchain confirmations take time (BTC: 10-60 minutes, ETH: 15-45 seconds, SOL: 5-10 seconds, etc.).

**Fee Model**: Platform fees use a hybrid model: `max(1% * amount, minimumFee)`. Project fees (optional) are added on top if configured. Minimum fees are enforced per currency to ensure profitability.

```javascript
// Withdraw from wallet
POST /api/wallet/507f1f77bcf86cd799439015/withdraw
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "toAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "amount": 0.01,
  "feeRate": 10 // For BTC/LTC only
}

// Response (Returns Immediately - Async Processing)
{
  "success": true,
  "message": "Withdrawal is processing",
  "data": {
    "transactionId": "507f1f77bcf86cd799439027",
    "status": "queued",
    "amount": 0.01,
    "platformFee": 0.00012, // max(1% * 0.01, 0.00012 BTC minimum)
    "projectFee": 0.00005, // Optional: if project fee is configured
    "totalFee": 0.00017, // Platform fee + project fee
    "message": "Withdrawal is processing. Check transaction status for updates."
  }
}

// Check Transaction Status
GET /api/wallet/transactions/507f1f77bcf86cd799439027
Authorization: Bearer {user_token}

// Response (During Processing)
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439027",
    "status": "prefunding", // or "processing", "broadcasted"
    "mainTxStatus": "pending",
    "mainTxHash": null,
    "platformFee": 0.0005,
    "createdAt": "2024-01-15T10:35:00Z"
  }
}

// Response (After Confirmation)
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439027",
    "status": "confirmed",
    "mainTxHash": "abc123def456ghi789jkl012mno345pqr678stu901vwx234yz",
    "mainTxStatus": "confirmed",
    "refundTxHash": "xyz789abc123def456ghi789jkl012mno345pqr678",
    "refundStatus": "confirmed",
    "networkFee": 0.0001,
    "platformFee": 0.0005,
    "completedAt": "2024-01-15T10:45:00Z"
  }
}
```

**Transaction Status Flow**:
- `queued` → Transaction added to processing queue
- `prefunding` → Platform sending pre-fund (if needed)
- `processing` → User transaction being prepared
- `broadcasted` → Transaction broadcast to blockchain
- `confirmed` → Transaction confirmed on blockchain
- `completed` → All operations completed successfully
- `failed` → Transaction failed (check error field)
- `partial` → User tx succeeded but refund failed (CRITICAL - requires manual intervention)

### 8. Validate Address

```javascript
// Validate cryptocurrency address
POST /api/wallet/validate-address
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "currency": "BTC"
}

// Response
{
  "success": true,
  "data": {
    "isValid": true,
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "format": "bech32"
  }
}
```

### 9. Get Supported Currencies

```javascript
// Get list of supported currencies
GET /api/wallet/currencies
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": [
    {
      "code": "BTC",
      "name": "Bitcoin",
      "network": "mainnet"
    },
    {
      "code": "ETH",
      "name": "Ethereum",
      "network": "mainnet"
    },
    {
      "code": "BNB",
      "name": "Binance Coin",
      "network": "bsc"
    },
    {
      "code": "LTC",
      "name": "Litecoin",
      "network": "mainnet"
    },
    {
      "code": "SOL",
      "name": "Solana",
      "network": "mainnet"
    },
    {
      "code": "TRX",
      "name": "Tron",
      "network": "mainnet"
    },
    {
      "code": "USDT",
      "name": "Tether",
      "network": "ethereum"
    }
  ]
}
```

---

## Billing & Payment Gateway Workflows

### 1. Setup Payment Gateway

```javascript
// Create payment gateway account
POST /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "provider": "paystack", // or "flutterwave", "monnify", "interswitch"
  "publicKey": "pk_test_abc123...",
  "secretKey": "[SECRET_KEY_NOT_RETURNED]",
  "config": {
    "currency": "NGN",
    "webhookUrl": "https://api.mudbase.com/api/billing/webhooks/paystack"
  }
}

// Response
{
  "message": "Payment gateway created successfully",
  "gateway": {
    "_id": "507f1f77bcf86cd799439017",
    "provider": "paystack",
    "status": "pending",
    "publicKey": "pk_test_abc123...",
    "createdAt": "2024-01-15T10:40:00Z"
  }
}

// Activate gateway
PATCH /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "status": "active"
}
```

### 2. Create Subscription Plan

```javascript
// Create a plan for your project
POST /api/billing/projects/507f1f77bcf86cd799439011/plans
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Pro Plan",
  "description": "Professional plan with advanced features",
  "pricing": {
    "type": "recurring",
    "monthly": {
      "amount": 5000, // in kobo (50 NGN)
      "currency": "NGN"
    },
    "yearly": {
      "amount": 50000, // in kobo (500 NGN)
      "currency": "NGN"
    },
    "usage": [
      {
        "metric": "api_calls",
        "unitPrice": 0.1, // per API call
        "includedUnits": 10000,
        "currency": "NGN"
      }
    ]
  },
  "features": [
    {
      "name": "api_access",
      "description": "Full API access",
      "included": true
    },
    {
      "name": "storage",
      "description": "100GB storage",
      "included": true,
      "limit": 107374182400 // 100GB in bytes
    }
  ],
  "limits": {
    "apiCalls": 100000,
    "storage": 107374182400,
    "bandwidth": 1073741824000
  },
  "trial": {
    "enabled": true,
    "days": 7
  },
  "isActive": true,
  "isDefault": false
}

// Response
{
  "message": "Plan created successfully",
  "plan": {
    "_id": "507f1f77bcf86cd799439018",
    "name": "Pro Plan",
    "description": "Professional plan with advanced features",
    "pricing": {
      "type": "recurring",
      "monthly": {
        "amount": 5000,
        "currency": "NGN"
      },
      "yearly": {
        "amount": 50000,
        "currency": "NGN"
      }
    },
    "features": [
      {
        "name": "api_access",
        "description": "Full API access",
        "included": true
      },
      {
        "name": "storage",
        "description": "100GB storage",
        "included": true,
        "limit": 107374182400
      }
    ],
    "isActive": true,
    "createdAt": "2024-01-15T10:40:00Z"
  }
}
```

### 3. Customer Checkout Flow

```javascript
// Step 1: Get available plans (Public API)
GET /api/billing/public/projects/507f1f77bcf86cd799439011/plans

// Response
{
  "plans": [
    {
      "_id": "507f1f77bcf86cd799439018",
      "name": "Pro Plan",
      "description": "Professional plan with advanced features",
      "pricing": {
        "monthly": {
          "amount": 5000,
          "currency": "NGN"
        },
        "yearly": {
          "amount": 50000,
          "currency": "NGN"
        }
      },
      "features": [ ... ]
    }
  ]
}

// Step 2: Create checkout session
POST /api/billing/public/projects/507f1f77bcf86cd799439011/checkout
Content-Type: application/json

{
  "planId": "507f1f77bcf86cd799439018",
  "billingCycle": "monthly",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  },
  "successUrl": "https://yourapp.com/billing/success",
  "cancelUrl": "https://yourapp.com/billing/cancel"
}

// Response
{
  "success": true,
  "data": {
    "checkoutUrl": "https://paystack.com/pay/abc123...",
    "reference": "mudbase_1705315200_abc123def456",
    "amount": 5000,
    "currency": "NGN"
  }
}

// Step 3: Redirect customer to authorizationUrl
// Customer completes payment on Paystack/Flutterwave

// Step 4: Payment Gateway redirects to callback URL
// Backend automatically verifies payment and creates subscription

// Step 5: Verify payment (if needed manually)
POST /api/billing/public/projects/507f1f77bcf86cd799439011/verify-payment
Content-Type: application/json

{
  "reference": "mudbase_1705315200_abc123def456",
  "orgId": "507f1f77bcf86cd799439013"
}

// Response
{
  "success": true,
  "message": "Payment verified and subscription created",
  "data": {
    "subscription": {
      "_id": "507f1f77bcf86cd799439019",
      "status": "active",
      "plan": {
        "_id": "507f1f77bcf86cd799439018",
        "name": "Pro Plan"
      },
      "customerEmail": "customer@example.com",
      "currentPeriodEnd": "2024-02-15T10:45:00Z",
      "billingCycle": "monthly"
    }
  }
}
```

### 4. Check Subscription Status

```javascript
// Check customer subscription (Public API)
GET /api/billing/public/projects/507f1f77bcf86cd799439011/subscription?email=customer@example.com

// Response
{
  "hasSubscription": true,
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active",
    "plan": {
      "_id": "507f1f77bcf86cd799439018",
      "name": "Pro Plan",
      "price": 5000,
      "currency": "NGN"
    },
    "customerEmail": "customer@example.com",
    "currentPeriodEnd": "2024-02-15T10:45:00Z",
    "billingCycle": "monthly",
    "createdAt": "2024-01-15T10:45:00Z"
  }
}
```

### 5. Check Feature Access

```javascript
// Check if customer has access to a feature
GET /api/billing/public/projects/507f1f77bcf86cd799439011/feature-access?email=customer@example.com&feature=api_access

// Response
{
  "hasAccess": true,
  "reason": "Active subscription",
  "plan": {
    "_id": "507f1f77bcf86cd799439018",
    "name": "Pro Plan"
  },
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active"
  }
}
```

### 6. Record Usage (Metered Billing)

```javascript
// Record usage for metered billing
POST /api/billing/public/projects/507f1f77bcf86cd799439011/usage
Content-Type: application/json

{
  "email": "customer@example.com",
  "metric": "api_calls",
  "quantity": 150
}

// Response
{
  "success": true,
  "message": "Usage recorded successfully"
}
```

### 7. Cancel Subscription

```javascript
// Cancel subscription
POST /api/billing/subscriptions/507f1f77bcf86cd799439019/cancel
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "cancelImmediately": false // Cancel at period end
}

// Response
{
  "message": "Subscription canceled successfully",
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active",
    "cancelAtPeriodEnd": true,
    "currentPeriodEnd": "2024-02-15T10:45:00Z",
    "canceledAt": "2024-01-15T11:00:00Z"
  }
}
```

### 8. Payment Gateway Webhook

```javascript
// Payment gateway sends webhook on payment events
POST /api/billing/webhooks/paystack
Content-Type: application/json
X-Paystack-Signature: {signature}

{
  "event": "charge.success",
  "data": {
    "reference": "mudbase_1705315200_abc123def456",
    "amount": 5000,
    "currency": "NGN",
    "customer": {
      "email": "customer@example.com"
    },
    "metadata": {
      "projectId": "507f1f77bcf86cd799439011",
      "planId": "507f1f77bcf86cd799439018",
      "billingCycle": "monthly",
      "orgId": "507f1f77bcf86cd799439013"
    }
  }
}

// Backend automatically:
// 1. Verifies webhook signature
// 2. Verifies payment
// 3. Creates subscription
// 4. Sends confirmation email
// 5. Triggers project webhook
```

---

## Database Operations

### 1. Create Collection Schema

```javascript
// Define collection schema
POST /api/projects/507f1f77bcf86cd799439011/schemas
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "products",
  "fields": [
    {
      "name": "name",
      "type": "string",
      "required": true,
      "indexed": true
    },
    {
      "name": "price",
      "type": "number",
      "required": true
    },
    {
      "name": "description",
      "type": "string"
    },
    {
      "name": "category",
      "type": "string",
      "indexed": true
    },
    {
      "name": "inStock",
      "type": "boolean",
      "default": true
    },
    {
      "name": "createdAt",
      "type": "date",
      "default": "now"
    }
  ],
  "indexes": [
    {
      "fields": ["category", "price"],
      "unique": false
    }
  ]
}

// Response
{
  "success": true,
  "collection": {
    "_id": "507f1f77bcf86cd799439020",
    "name": "products",
    "slug": "products",
    "project": "507f1f77bcf86cd799439011",
    "fields": [
      {
        "name": "name",
        "type": "string",
        "required": true,
        "indexed": true
      },
      {
        "name": "price",
        "type": "number",
        "required": true
      },
      {
        "name": "description",
        "type": "string"
      },
      {
        "name": "category",
        "type": "string",
        "indexed": true
      },
      {
        "name": "inStock",
        "type": "boolean",
        "default": true
      }
    ],
    "createdAt": "2024-01-15T10:50:00Z"
  }
}
```

### 2. Create Document

```javascript
// Create a document in collection
POST /api/projects/507f1f77bcf86cd799439011/data/products
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Laptop",
  "price": 999.99,
  "description": "High-performance laptop",
  "category": "electronics",
  "inStock": true
}

// Response
{
  "message": "Data created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439021",
    "name": "Laptop",
    "price": 999.99,
    "description": "High-performance laptop",
    "category": "electronics",
    "inStock": true,
    "createdAt": "2024-01-15T10:55:00Z",
    "updatedAt": "2024-01-15T10:55:00Z"
  }
}
```

### 3. Query Documents

```javascript
// Get all documents with filters
GET /api/projects/507f1f77bcf86cd799439011/data/products?category=electronics&price[gte]=500&limit=10&page=1
Authorization: Bearer {user_token}

// Response
{
  "data": [
    {
      "_id": "507f1f77bcf86cd799439021",
      "name": "Laptop",
      "price": 999.99,
      "description": "High-performance laptop",
      "category": "electronics",
      "inStock": true,
      "createdAt": "2024-01-15T10:55:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 1,
    "totalPages": 1
  }
}

// Advanced query
POST /api/projects/507f1f77bcf86cd799439011/data/products/query
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "filter": {
    "$and": [
      { "category": "electronics" },
      { "price": { "$gte": 500 } },
      { "inStock": true }
    ]
  },
  "sort": { "price": -1 },
  "limit": 10,
  "skip": 0
}
```

### 4. Update Document

```javascript
// Update document
PATCH /api/projects/507f1f77bcf86cd799439011/data/products/507f1f77bcf86cd799439021
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "price": 899.99,
  "inStock": false
}

// Response
{
  "message": "Data updated successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439021",
    "name": "Laptop",
    "price": 899.99,
    "description": "High-performance laptop",
    "category": "electronics",
    "inStock": false,
    "createdAt": "2024-01-15T10:55:00Z",
    "updatedAt": "2024-01-15T11:00:00Z"
  }
}
```

### 5. Delete Document

```javascript
// Delete document
DELETE /api/projects/507f1f77bcf86cd799439011/data/products/507f1f77bcf86cd799439021
Authorization: Bearer {user_token}

// Response
{
  "message": "Data deleted successfully"
}
```

---

## Real-time Chat System

### 1. Create Chat

```javascript
// Create a new chat
POST /api/projects/507f1f77bcf86cd799439011/chats
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Team Discussion",
  "type": "group", // or "direct"
  "participants": [
    "507f1f77bcf86cd799439012",
    "507f1f77bcf86cd799439022"
  ],
  "metadata": {
    "projectId": "507f1f77bcf86cd799439011"
  }
}

// Response
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439023",
    "name": "Team Discussion",
    "type": "group",
    "participants": [
      {
        "user": "507f1f77bcf86cd799439012",
        "role": "admin",
        "joinedAt": "2024-01-15T11:05:00Z"
      },
      {
        "user": "507f1f77bcf86cd799439022",
        "role": "member",
        "joinedAt": "2024-01-15T11:05:00Z"
      }
    ],
    "createdBy": "507f1f77bcf86cd799439012",
    "project": "507f1f77bcf86cd799439011",
    "createdAt": "2024-01-15T11:05:00Z"
  }
}
```

### 2. Send Message (HTTP)

```javascript
// Send message via HTTP
POST /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "content": "Hello team! How's everyone doing?",
  "type": "text" // or "image", "file", etc.
}

// Response
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439024",
    "content": "Hello team! How's everyone doing?",
    "type": "text",
    "sender": "507f1f77bcf86cd799439012",
    "chat": "507f1f77bcf86cd799439023",
    "project": "507f1f77bcf86cd799439011",
    "createdAt": "2024-01-15T11:06:00Z",
    "updatedAt": "2024-01-15T11:06:00Z"
  }
}
```

### 3. Real-time Messaging (Socket.IO)

```javascript
// Client-side Socket.IO connection
import io from 'socket.io-client';

const socket = io('https://api.mudbase.com', {
  auth: {
    token: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
  }
});

// Join chat room
socket.emit('chat:join', {
  chatId: '507f1f77bcf86cd799439023',
  projectId: '507f1f77bcf86cd799439011'
});

// Send message via Socket.IO
socket.emit('chat:message:send', {
  chatId: '507f1f77bcf86cd799439023',
  content: 'Hello from real-time!',
  type: 'text'
});

// Listen for new messages
socket.on('chat:message:new', (data) => {
  console.log('New message:', data);
  // {
  //   message: {
  //     _id: '507f1f77bcf86cd799439025',
  //     content: 'Hello from real-time!',
  //     sender: { ... },
  //     createdAt: '2024-01-15T11:07:00Z'
  //   },
  //   chatId: '507f1f77bcf86cd799439023'
  // }
});

// Typing indicator
socket.emit('chat:typing', {
  chatId: '507f1f77bcf86cd799439023',
  isTyping: true
});

socket.on('chat:typing', (data) => {
  console.log('User typing:', data);
  // {
  //   userId: '507f1f77bcf86cd799439012',
  //   chatId: '507f1f77bcf86cd799439023',
  //   isTyping: true
  // }
});

// Voice/Video call events
socket.emit('chat:call:initiate', {
  chatId: '507f1f77bcf86cd799439023',
  type: 'video' // or 'voice'
});

socket.on('chat:call:incoming', (data) => {
  console.log('Incoming call:', data);
});

socket.emit('chat:call:accept', {
  callId: 'call_abc123'
});

socket.emit('chat:call:reject', {
  callId: 'call_abc123'
});

socket.emit('chat:call:end', {
  callId: 'call_abc123'
});
```

### 4. Edit Message

```javascript
// Edit message
PATCH /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "content": "Hello team! How's everyone doing? (edited)"
}

// Socket.IO event also emitted: 'chat:message:updated'
```

### 5. Delete Message

```javascript
// Delete message
DELETE /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024
Authorization: Bearer {user_token}

// Socket.IO event also emitted: 'chat:message:deleted'
```

### 6. Add Reaction

```javascript
// Add reaction to message
POST /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024/reactions
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "emoji": "👍"
}

// Socket.IO event also emitted: 'chat:message:reaction'
```

---

## Integration System

### 1. Create Integration

```javascript
// Create integration
POST /api/projects/507f1f77bcf86cd799439011/integrations
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Twitter API Integration",
  "provider": "twitter",
  "category": "social",
    "settings": {
      "isActive": true
      // NOTE: API keys, secrets, and tokens are encrypted and never returned in responses
    },
  "config": {
    "rateLimit": 100,
    "timeout": 5000
  }
}

// Response
{
  "integration": {
    "_id": "507f1f77bcf86cd799439026",
    "name": "Twitter API Integration",
    "provider": "twitter",
    "category": "social",
    "project": "507f1f77bcf86cd799439011",
    "settings": {
      "isActive": true
    },
    "createdAt": "2024-01-15T11:10:00Z",
    "updatedAt": "2024-01-15T11:10:00Z"
  }
}
```

### 2. Execute Integration

```javascript
// Execute integration
POST /api/projects/507f1f77bcf86cd799439011/integrations/507f1f77bcf86cd799439026/execute
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "method": "POST",
  "endpoint": "/2/tweets",
  "body": {
    "text": "Hello from MUDBASE!"
  },
  "headers": {
    "Content-Type": "application/json"
  }
}

// Response
{
  "success": true,
  "data": {
    "status": 201,
    "data": {
      "id": "1234567890",
      "text": "Hello from MUDBASE!",
      "created_at": "2024-01-15T11:12:00Z"
    },
    "headers": {
      "content-type": "application/json",
      "x-rate-limit-remaining": "299"
    }
  },
  "usage": {
    "apiCalls": 1,
    "timestamp": "2024-01-15T11:12:00Z"
  }
}
```

### 3. Get Integration Usage Stats

```javascript
// Get usage statistics
GET /api/projects/507f1f77bcf86cd799439011/integrations/507f1f77bcf86cd799439026/usage?period=month
Authorization: Bearer {user_token}

// Response
{
  "stats": {
    "totalCalls": 1250,
    "successfulCalls": 1200,
    "failedCalls": 50,
    "successRate": 96.0,
    "period": "month",
    "breakdown": [
      {
        "date": "2024-01-15",
        "calls": 45,
        "successful": 43,
        "failed": 2
      }
    ]
  }
}
```

---

## Complete End-to-End Scenarios

### Scenario 1: E-commerce Platform Setup

```javascript
// Step 1: Register Admin User
POST /api/auth/local/register
{
  "email": "admin@ecommerce.com",
  "password": "SecurePass123!",
  "firstName": "Admin",
  "lastName": "User"
}

// Step 2: Create Organization
POST /api/orgs
Authorization: Bearer {admin_token}
{
  "name": "Ecommerce Corp"
}

// Step 3: Create Project
POST /api/projects
Authorization: Bearer {admin_token}
{
  "name": "Ecommerce Platform",
  "orgId": "{org_id}"
}

// Step 4: Setup Payment Gateway
POST /api/billing/orgs/{org_id}/payment-gateway
Authorization: Bearer {admin_token}
{
  "provider": "paystack",
  "publicKey": "pk_test_...",
  "secretKey": "sk_test_..."
}

// Step 5: Create Product Schema
POST /api/projects/{project_id}/schemas
Authorization: Bearer {admin_token}
{
  "name": "products",
  "fields": [
    { "name": "name", "type": "string", "required": true },
    { "name": "price", "type": "number", "required": true },
    { "name": "stock", "type": "number", "required": true }
  ]
}

// Step 6: Create Order Schema
POST /api/projects/{project_id}/schemas
Authorization: Bearer {admin_token}
{
  "name": "orders",
  "fields": [
    { "name": "customerId", "type": "string", "required": true },
    { "name": "products", "type": "array", "required": true },
    { "name": "total", "type": "number", "required": true },
    { "name": "status", "type": "string", "default": "pending" }
  ]
}

// Step 7: Customer Registration
POST /api/auth/local/register
{
  "email": "customer@example.com",
  "password": "CustomerPass123!",
  "firstName": "John",
  "lastName": "Customer",
  "projectId": "{project_id}"
}

// Step 8: Customer Creates Order
POST /api/projects/{project_id}/data/orders
Authorization: Bearer {customer_token}
{
  "customerId": "{customer_id}",
  "products": [
    { "productId": "{product_id}", "quantity": 2 }
  ],
  "total": 1999.98,
  "status": "pending"
}

// Step 9: Process Payment
POST /api/billing/public/projects/{project_id}/checkout
{
  "planId": "{plan_id}",
  "billingCycle": "one-time",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  },
  "amount": 1999.98
}

// Step 10: Update Order Status
PATCH /api/projects/{project_id}/data/orders/{order_id}
Authorization: Bearer {admin_token}
{
  "status": "paid"
}
```

### Scenario 2: Crypto Wallet Integration

```javascript
// Step 1: User generates Bitcoin wallet key
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
{
  "currency": "BTC"
}

// Response includes privateKey (shown once only) that user must save securely

// Step 2: User creates wallet with generated key
POST /api/wallet/create
Authorization: Bearer {user_token}
{
  "currency": "BTC",
  "privateKey": "[PRIVATE_KEY_FROM_STEP_1]",
  "address": "[ADDRESS_FROM_STEP_1]"
}

// Response does NOT include privateKey - it's encrypted and stored securely

// Step 3: Check wallet balance
GET /api/wallet/{wallet_id}/balance
Authorization: Bearer {user_token}

// Step 4: Receive payment (external)
// Bitcoin sent to wallet address

// Step 5: Withdraw funds (async processing)
POST /api/wallet/{wallet_id}/withdraw
Authorization: Bearer {user_token}
{
  "toAddress": "bc1q...",
  "amount": 0.01,
  "feeRate": 10
}

// Response (returns immediately - async processing)
{
  "success": true,
  "data": {
    "transactionId": "...",
    "status": "queued",
    "platformFee": 0.0005 // max(1% * 0.01, 0.0005 BTC minimum)
  }
}

// Step 6: Check transaction status (poll until confirmed)
GET /api/wallet/transactions/{transaction_id}
Authorization: Bearer {user_token}

// Response (after confirmation - may take 10-60 minutes for BTC)
{
  "success": true,
  "data": {
    "status": "confirmed",
    "mainTxHash": "...",
    "mainTxStatus": "confirmed"
  }
}
```

### Scenario 3: SaaS Subscription Flow

```javascript
// Step 1: Setup billing plan
POST /api/billing/projects/{project_id}/plans
Authorization: Bearer {admin_token}
{
  "name": "Premium",
  "pricing": {
    "monthly": { "amount": 10000, "currency": "NGN" },
    "yearly": { "amount": 100000, "currency": "NGN" }
  },
  "features": [
    { "name": "api_access", "included": true },
    { "name": "storage", "included": true, "limit": 107374182400 }
  ]
}

// Step 2: Customer subscribes
POST /api/billing/public/projects/{project_id}/checkout
{
  "planId": "{plan_id}",
  "billingCycle": "monthly",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  }
}

// Step 3: Check subscription status
GET /api/billing/public/projects/{project_id}/subscription?email=customer@example.com

// Step 4: Check feature access
GET /api/billing/public/projects/{project_id}/feature-access?email=customer@example.com&feature=api_access

// Step 5: Record usage
POST /api/billing/public/projects/{project_id}/usage
{
  "email": "customer@example.com",
  "metric": "api_calls",
  "quantity": 150
}
```

---

## API Reference Quick Guide

### Authentication Endpoints

```
POST   /api/auth/register                - Register new user (org-based)
POST   /api/auth/login                   - Login user (org-based)
POST   /api/auth/logout                  - Logout user (org-based)
GET    /api/auth/session                 - Get current session (org-based)
POST   /api/auth/password-reset          - Request password reset (org-based)
POST   /api/auth/password-reset/{token}  - Reset password (org-based)
POST   /api/auth/local/register          - Register new user (project-based)
POST   /api/auth/local/login             - Login user (project-based)
POST   /api/auth/local/logout            - Logout user (project-based)
GET    /api/auth/local/session           - Get current session (project-based)
POST   /api/auth/local/password-reset    - Request password reset (project-based)
POST   /api/auth/local/password-reset/{token} - Reset password (project-based)
GET    /api/auth/oauth/{provider}        - Initiate OAuth
GET    /api/auth/oauth/{provider}/callback - OAuth callback
POST   /api/auth/magic-link/send         - Send magic link
GET    /api/auth/magic-link/verify        - Verify magic link
POST   /api/auth/otp/send                - Send OTP
POST   /api/auth/otp/verify              - Verify OTP
POST   /api/auth/2fa/setup               - Setup 2FA
POST   /api/auth/2fa/verify              - Verify 2FA
```

### Wallet Endpoints

```
GET    /api/wallet/currencies            - Get supported currencies
POST   /api/wallet/generate-key          - Generate key pair
POST   /api/wallet/validate-address      - Validate address
POST   /api/wallet/create                 - Create wallet
GET    /api/wallet                        - Get user wallets
GET    /api/wallet/{id}/balance          - Get wallet balance
GET    /api/wallet/{id}/private-key      - Get wallet private key (WARNING: Sensitive)
POST   /api/wallet/{id}/withdraw          - Withdraw funds
```

### Billing Endpoints

```
POST   /api/billing/orgs/{orgId}/payment-gateway - Create gateway
GET    /api/billing/orgs/{orgId}/payment-gateway - Get gateway status
POST   /api/billing/projects/{id}/plans         - Create plan
GET    /api/billing/projects/{id}/plans         - Get plans
GET    /api/billing/public/projects/{id}/plans  - Get public plans
POST   /api/billing/public/projects/{id}/checkout - Create checkout
POST   /api/billing/public/projects/{id}/verify-payment - Verify payment
GET    /api/billing/public/projects/{id}/subscription - Check subscription
GET    /api/billing/public/projects/{id}/feature-access - Check feature access
POST   /api/billing/public/projects/{id}/usage   - Record usage
POST   /api/billing/subscriptions/{id}/cancel   - Cancel subscription
POST   /api/billing/webhooks/{provider}         - Payment webhook
```

### Database Endpoints

```
POST   /api/projects/{id}/schemas         - Create schema
GET    /api/projects/{id}/schemas         - Get schemas
POST   /api/projects/{id}/data/{collection} - Create document
GET    /api/projects/{id}/data/{collection} - Query documents
PATCH  /api/projects/{id}/data/{collection}/{id} - Update document
DELETE /api/projects/{id}/data/{collection}/{id} - Delete document
POST   /api/projects/{id}/data/{collection}/query - Advanced query
```

### Chat Endpoints

```
POST   /api/projects/{id}/chats            - Create chat
GET    /api/projects/{id}/chats           - Get user chats
POST   /api/projects/{id}/chats/{id}/messages - Send message
GET    /api/projects/{id}/chats/{id}/messages - Get messages
PATCH  /api/projects/{id}/chats/{id}/messages/{id} - Edit message
DELETE /api/projects/{id}/chats/{id}/messages/{id} - Delete message
POST   /api/projects/{id}/chats/{id}/messages/{id}/reactions - Add reaction
```

### Integration Endpoints

```
GET    /api/projects/{id}/integrations/templates - Get templates
POST   /api/projects/{id}/integrations          - Create integration
GET    /api/projects/{id}/integrations          - Get integrations
POST   /api/projects/{id}/integrations/{id}/test - Test integration
POST   /api/projects/{id}/integrations/{id}/execute - Execute integration
GET    /api/projects/{id}/integrations/{id}/usage - Get usage stats
```

---

## Security Features

### 1. Session Management
- **Idle Timeout**: 30 minutes of inactivity
- **Rolling Sessions**: Session extends on activity
- **Secure Cookies**: HttpOnly, Secure in production

### 2. Access Control
- **RBAC**: Role-Based Access Control (owner, admin, developer, viewer)
- **Project-Level Access**: Users can only access their project's resources
- **Organization-Level Access**: Users can only access their org's resources

### 3. Encryption
- **Private Keys**: AES-256-GCM encryption at rest
- **API Credentials**: Encrypted in database
- **Password Hashing**: bcrypt with salt rounds 12

### 4. Rate Limiting
- **Global**: 100 requests per 15 minutes per IP
- **Authentication**: 5 login attempts per 15 minutes
- **API Keys**: Configurable per project

### 5. Audit Logging
- All authentication events logged
- All authorization failures logged
- All sensitive operations logged

---

## Best Practices

### 1. Authentication
- Always use HTTPS in production
- Store JWT tokens securely (httpOnly cookies recommended)
- Implement token refresh mechanism
- Use 2FA for sensitive accounts

### 2. Wallet Management
- Never log private keys
- Always validate addresses before transactions
- Use custom keys only when necessary
- Keep backup of private keys securely
- **Async Processing**: Withdrawals are processed asynchronously - check transaction status for updates
- **Real Confirmations**: System uses real blockchain confirmations (not fake delays)
- **Minimum Fees**: Enforced per currency to ensure profitability
- **Status Tracking**: Monitor transaction status through: `queued` → `prefunding` → `processing` → `broadcasted` → `confirmed` → `completed`

### 3. Billing
- Verify webhook signatures
- Handle payment failures gracefully
- Implement retry logic for failed payments
- Monitor subscription status

### 4. Database
- Use indexes for frequently queried fields
- Implement pagination for large datasets
- Validate input data
- Use transactions for critical operations

### 5. Real-time
- Handle connection failures
- Implement reconnection logic
- Use rooms for efficient message delivery
- Clean up on disconnect

---

## Error Handling

### Common Error Codes

```javascript
// Authentication Errors
401 - Unauthorized (Invalid token)
403 - Forbidden (Insufficient permissions)
429 - Too Many Requests (Rate limit exceeded)

// Validation Errors
400 - Bad Request (Invalid input)
422 - Unprocessable Entity (Validation failed)

// Resource Errors
404 - Not Found (Resource doesn't exist)
409 - Conflict (Resource already exists)

// Server Errors
500 - Internal Server Error
503 - Service Unavailable
```

### Error Response Format

```javascript
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "email": "Email is required",
      "password": "Password must be at least 8 characters"
    }
  }
}
```

---

## Project Fee Management

### Overview

Project owners can configure their own fees per currency, which are collected in addition to platform fees. These fees are accumulated and paid out via automated bi-weekly payouts.

### 1. Configure Project Fee Settings

```javascript
// Step 1: Create or update fee settings
POST /api/projects/507f1f77bcf86cd799439011/fee-settings
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "enabled": true,
  "feeAmount": 0.00005, // Must be < platform minimum fee (0.00012 BTC)
  "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "payoutThreshold": 0.001 // Minimum amount before payout
}

// Response
{
  "success": true,
  "message": "Fee settings updated successfully",
  "data": {
    "feeSettings": {
      "BTC": {
        "enabled": true,
        "feeAmount": 0.00005,
        "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "payoutThreshold": 0.001,
        "addressVerified": false
      }
    }
  }
}
```

### 2. Verify Payout Address

```javascript
// Step 1: Initiate address verification
POST /api/projects/507f1f77bcf86cd799439011/fee-settings/BTC/verify-address
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "message": "Test transaction sent. Please confirm receipt or provide transaction hash from the address.",
  "data": {
    "verificationStatus": "pending",
    "testTxHash": "abc123def456...",
    "testAmount": 0.00001,
    "instructions": "Either confirm you received the test transaction, or send a transaction FROM the payout address to prove ownership."
  }
}

// Step 2: Confirm verification
POST /api/projects/507f1f77bcf86cd799439011/fee-settings/BTC/confirm-verification
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "txHash": "xyz789abc123..." // Transaction hash proving ownership
}

// Response
{
  "success": true,
  "message": "Address verified successfully",
  "data": {
    "verified": true,
    "verifiedAt": "2024-01-15T12:00:00Z"
  }
}
```

### 3. Check Fee Balance

```javascript
// Get balance for specific currency
GET /api/projects/507f1f77bcf86cd799439011/fee-balances/BTC
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "currency": "BTC",
    "collectedAmount": 0.0005,
    "threshold": 0.001,
    "status": "accumulating",
    "nextScheduledPayoutDate": null,
    "lastPayoutDate": null,
    "totalPaidOut": 0,
    "totalCollected": 0.0005
  }
}

// Get all balances
GET /api/projects/507f1f77bcf86cd799439011/fee-balances
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "balances": [
      {
        "currency": "BTC",
        "collectedAmount": 0.0005,
        "threshold": 0.001,
        "status": "accumulating"
      },
      {
        "currency": "ETH",
        "collectedAmount": 0.002,
        "threshold": 0.001,
        "status": "ready",
        "nextScheduledPayoutDate": "2024-01-17T02:00:00Z"
      }
    ]
  }
}
```

### 4. View Payout History

```javascript
// Get payout history
GET /api/projects/507f1f77bcf86cd799439011/payout-history?limit=10&offset=0
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "payouts": [
      {
        "_id": "507f1f77bcf86cd799439030",
        "currency": "BTC",
        "grossAmount": 0.001,
        "networkFee": 0.0001,
        "netAmount": 0.0009,
        "toAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "txHash": "abc123def456...",
        "status": "completed",
        "scheduledAt": "2024-01-15T02:00:00Z",
        "processedAt": "2024-01-15T02:05:00Z",
        "confirmedAt": "2024-01-15T02:45:00Z",
        "createdAt": "2024-01-15T02:00:00Z"
      }
    ],
    "total": 1,
    "limit": 10,
    "offset": 0
  }
}
```

### 5. Request Manual Payout

```javascript
// Request manual payout (restricted: once per 30 days per currency)
POST /api/projects/507f1f77bcf86cd799439011/payouts/request-manual
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC"
}

// Response
{
  "success": true,
  "message": "Manual payout requested and processed",
  "data": {
    "payout": {
      "_id": "507f1f77bcf86cd799439031",
      "currency": "BTC",
      "netAmount": 0.0009,
      "txHash": "abc123def456...",
      "status": "completed"
    }
  }
}
```

### 6. Fee Dashboard

```javascript
// Get comprehensive fee dashboard
GET /api/projects/507f1f77bcf86cd799439011/fee-dashboard
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "feeSettings": {
      "BTC": {
        "enabled": true,
        "feeAmount": 0.00005,
        "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "addressVerified": true
      }
    },
    "balances": [
      {
        "currency": "BTC",
        "collectedAmount": 0.0005,
        "threshold": 0.001,
        "status": "accumulating",
        "totalPaidOut": 0.0009,
        "totalCollected": 0.0014
      }
    ],
    "recentPayouts": [
      {
        "_id": "507f1f77bcf86cd799439030",
        "currency": "BTC",
        "netAmount": 0.0009,
        "status": "completed",
        "createdAt": "2024-01-15T02:00:00Z"
      }
    ],
    "totalEarned": 0.0009
  }
}
```

### Important Notes

1. **Fee Validation**: Project fee must be less than platform minimum fee (cannot undercut platform)
2. **Address Verification**: Required before fees are collected (prevents fraud)
3. **Automated Payouts**: Bi-weekly schedule (Wed/Sat 2 AM UTC) when threshold is met
4. **Network Fees**: Deducted from payout amount (platform pays network fees)
5. **Manual Payouts**: Restricted to once per 30 days per currency, requires 25% of threshold
6. **Fee Collection**: Automatically collected during user withdrawals if enabled and verified

---

## Conclusion

This document provides a comprehensive overview of the MUDBASE Backend-as-a-Service platform, including:

- **Multiple Authentication Methods**: Local, OAuth, Magic Link, OTP, 2FA
- **Wallet as a Service**: Support for 7 cryptocurrencies with custom key support
- **Project Fee System**: Project owners can set their own fees with automated bi-weekly payouts
- **Billing System**: Nigerian payment gateways (Paystack, Flutterwave) with subscription management
- **Real-time Features**: Socket.IO-based chat system
- **Database Operations**: Flexible schema-based collections
- **Integration System**: 50+ API integrations
- **Security**: RBAC, encryption, rate limiting, audit logging

For more detailed API documentation, refer to the OpenAPI specification at `/api-docs`.


## Table of Contents
1. [System Architecture Overview](#system-architecture-overview)
2. [Authentication Flows](#authentication-flows)
3. [Organization & Project Setup](#organization--project-setup)
4. [Wallet as a Service Workflows](#wallet-as-a-service-workflows)
5. [Billing & Payment Gateway Workflows](#billing--payment-gateway-workflows)
6. [Database Operations](#database-operations)
7. [Real-time Chat System](#real-time-chat-system)
8. [Integration System](#integration-system)
9. [Complete End-to-End Scenarios](#complete-end-to-end-scenarios)
10. [API Reference Quick Guide](#api-reference-quick-guide)

---

## System Architecture Overview

### Core Components

```
┌────────────────────────────────────────────────────────────┐
│                    MUDBASE Backend Platform                │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Auth Layer  │  │  API Layer   │  │  Socket.IO   │      │
│  │              │  │              │  │  Real-time   │      │
│  │ • Local      │  │ • REST API   │  │ • Chat       │      │
│  │ • OAuth      │  │ • Webhooks   │  │ • Events     │      │
│  │ • Magic Link │  │ • GraphQL    │  │ • Database   │      │
│  │ • OTP        │  │              │  │              │      │
│  │ • 2FA        │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Database   │  │   Storage    │  │  Integrations│      │
│  │              │  │              │  │              │      │
│  │ • MongoDB    │  │ • File Store │  │ • 50+ APIs   │      │
│  │ • Collections│  │ • Buckets    │  │ • Webhooks   │      │
│  │ • Indexes    │  │ • CDN        │  │ • Custom     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Wallet     │  │   Billing    │  │  Security    │      │
│  │   Service    │  │              │  │              │      │
│  │              │  │ • Paystack   │  │ • RBAC       │      │
│  │ • BTC        │  │ • Flutterwave│  │ • Encryption │      │
│  │ • ETH        │  │ • Plans      │  │ • Rate Limit│       │
│  │ • SOL        │  │ • Usage      │  │ • Audit Log  │      │
│  │ • TRX        │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└────────────────────────────────────────────────────────────┘
```

### Data Flow

```
User Request → Authentication → Authorization (RBAC) → Business Logic → Database/External API → Response
                ↓
            JWT Token
                ↓
        Session Management (30min idle timeout)
                ↓
        Real-time Updates via Socket.IO
```

---

## Authentication Flows

### 1. Basic Authentication (Organization-based)

These endpoints are for organization-level authentication and create organizations automatically.

#### Registration Flow

```javascript
// Step 1: User Registration (creates organization automatically)
POST /api/auth/register
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "firstName": "John",
  "lastName": "Doe",
  "orgName": "Acme Corp" // Optional
}

// Response
{
  "message": "User registered successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner",
    "emailVerified": false,
    "org": "507f1f77bcf86cd799439013"
  }
}

// Step 2: Login
POST /api/auth/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]"
}

// Response
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  }
}

// Step 3: Get Session
GET /api/auth/session
Authorization: Bearer {token}

// Response
{
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner"
  },
  "authenticated": true
}

// Step 4: Password Reset Request
POST /api/auth/password-reset
Content-Type: application/json

{
  "email": "john.doe@example.com"
}

// Response
{
  "message": "Password reset email sent"
}

// Step 5: Reset Password
POST /api/auth/password-reset/{token}
Content-Type: application/json

{
  "password": "[NEW_PASSWORD]"
}

// Response
{
  "message": "Password reset successful"
}
```

### 2. Local Authentication (Project-based)

These endpoints are for project-level authentication and require a projectId.

#### Registration Flow

```javascript
// Step 1: User Registration
POST /api/auth/local/register
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "firstName": "John",
  "lastName": "Doe",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "message": "User registered successfully",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "emailVerified": false,
    "role": "developer",
    "org": "507f1f77bcf86cd799439013"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

// Step 2: Email Verification (Optional)
POST /api/auth/verify-email
Authorization: Bearer {token}
Content-Type: application/json

{
  "token": "email-verification-token-from-email"
}

// Response
{
  "success": true,
  "message": "Email verified successfully"
}

// Step 3: Login
POST /api/auth/local/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  },
  "expiresIn": "24h"
}
```

#### Password Reset (Project-based)

```javascript
// Step 1: Request password reset
POST /api/auth/local/password-reset
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "projectId": "507f1f77bcf86cd799439011" // Optional
}

// Response
{
  "message": "Password reset email sent"
}

// Step 2: Reset password
POST /api/auth/local/password-reset/{token}
Content-Type: application/json

{
  "password": "[NEW_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011" // Optional
}

// Response
{
  "message": "Password reset successful"
}
```

#### Session Management

```javascript
// Session expires after 30 minutes of inactivity
// Server automatically extends session on activity (rolling: true)

// Check current session (organization-based)
GET /api/auth/session
Authorization: Bearer {token}

// Check current session (project-based)
GET /api/auth/local/session?projectId={projectId}
Authorization: Bearer {token}

// Response
{
  "user": {
    "id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "project": {
      "id": "507f1f77bcf86cd799439011",
      "name": "My Awesome App",
      "role": "developer"
    }
  },
  "authenticated": true
}

// Response
{
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  }
}

// Refresh token (if implemented)
POST /api/auth/refresh
Authorization: Bearer {token}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": "24h"
}

// Logout (organization-based)
POST /api/auth/logout
Authorization: Bearer {token}

// Response
{
  "message": "Logout successful"
}

// Logout (project-based)
POST /api/auth/local/logout
Authorization: Bearer {token}

// Response
{
  "message": "Logout successful"
}
```

**Key Differences Between Organization-based and Project-based Auth:**

- **Organization-based** (`/api/auth/*`): Creates organizations automatically, simpler flow, good for admin/owner accounts
- **Project-based** (`/api/auth/local/*`): Requires existing project, project-specific authentication, includes rate limiting and captcha verification, better for end-user authentication

### 3. OAuth Authentication

#### Supported Providers
- Google, GitHub, Facebook, Microsoft, Apple, LinkedIn, Discord, Twitter, etc.

#### OAuth Flow Example (Google)

```javascript
// Step 1: Initiate OAuth
// Redirect user to:
GET /api/auth/oauth/google?projectId=507f1f77bcf86cd799439011

// User is redirected to Google consent screen
// After consent, Google redirects to:
GET /api/auth/oauth/google/callback?code={authorization_code}

// Step 2: Backend processes OAuth callback
// Backend automatically:
// 1. Exchanges code for access token
// 2. Fetches user profile from Google
// 3. Creates/updates user in database
// 4. Generates JWT token
// 5. Redirects to frontend with token

// Frontend receives redirect:
https://yourapp.com/auth/callback?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// Step 3: Use token for authenticated requests
Authorization: Bearer {token}

// Response (from OAuth callback redirect)
// Frontend receives:
https://yourapp.com/auth/callback?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...&user={"id":"507f1f77bcf86cd799439012","email":"john.doe@example.com"}
```

#### OAuth Provider Configuration

```javascript
// Configure OAuth provider in project settings
PATCH /api/projects/{projectId}
Authorization: Bearer {admin_token}
Content-Type: application/json

{
  "authProviders": {
    "google": {
      "enabled": true,
      "clientId": "your-google-client-id",
      "clientSecret": "your-google-client-secret",
      "callbackUrl": "https://api.mudbase.com/api/auth/oauth/google/callback"
    },
    "github": {
      "enabled": true,
      "clientId": "your-github-client-id",
      "clientSecret": "your-github-client-secret"
    }
  }
}
```

### 4. Magic Link Authentication

```javascript
// Step 1: Request Magic Link
POST /api/auth/magic-link/send
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "projectId": "507f1f77bcf86cd799439011",
  "redirectUrl": "https://yourapp.com/auth/verify"
}

// Response
{
  "success": true,
  "message": "Magic link sent to email"
}

// Step 2: User clicks link in email
// Email contains: https://api.mudbase.com/api/auth/magic-link/verify?token={magic_token}

// Step 3: Verify Magic Link
GET /api/auth/magic-link/verify?token={magic_token}&projectId={projectId}

// Response (redirects to frontend with token)
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

### 5. OTP Authentication (SMS/Email)

```javascript
// Step 1: Request OTP
POST /api/auth/otp/send
Content-Type: application/json

{
  "identifier": "john.doe@example.com", // or phone number
  "method": "email", // or "sms" or "auto"
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "message": "OTP sent to email",
  "expiresIn": 300 // 5 minutes
}

// Step 2: Verify OTP
POST /api/auth/otp/verify
Content-Type: application/json

{
  "identifier": "john.doe@example.com",
  "otp": "123456",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

### 6. Two-Factor Authentication (2FA)

```javascript
// Step 1: Enable 2FA
POST /api/auth/2fa/setup
Authorization: Bearer {token}
Content-Type: application/json

// Response
{
  "secret": "JBSWY3DPEHPK3PXP",
  "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..."
}

// Step 2: Verify and Enable
POST /api/auth/2fa/verify
Authorization: Bearer {token}
Content-Type: application/json

{
  "token": "123456" // TOTP code from authenticator app
}

// Step 3: Login with 2FA
POST /api/auth/local/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response (requires 2FA)
{
  "requires2FA": true,
  "tempToken": "temp-token-for-2fa-verification"
}

// Step 4: Verify 2FA
POST /api/auth/2fa/verify-login
Content-Type: application/json

{
  "tempToken": "temp-token-for-2fa-verification",
  "token": "123456"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

---

## Organization & Project Setup

### Complete Setup Workflow

```javascript
// Step 1: Create Organization
POST /api/orgs
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Acme Corporation",
  "slug": "acme-corp" // auto-generated if not provided
}

// Response
{
  "success": true,
  "org": {
    "_id": "507f1f77bcf86cd799439013",
    "name": "Acme Corporation",
    "slug": "acme-corp",
    "members": ["507f1f77bcf86cd799439012"],
    "createdAt": "2024-01-15T10:00:00Z"
  }
}

// Step 2: Create Project
POST /api/projects
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "My Awesome App",
  "orgId": "507f1f77bcf86cd799439013",
  "description": "A revolutionary app"
}

// Response
{
  "success": true,
  "project": {
    "_id": "507f1f77bcf86cd799439011",
    "name": "My Awesome App",
    "slug": "my-awesome-app",
    "org": "507f1f77bcf86cd799439013",
    "apiKey": "pk_live_abc123...",
    // NOTE: secretKey is NEVER returned in responses
    "settings": {
      "auth": {
        "requireEmailVerification": true,
        "sessionTimeout": 1800000
      }
    }
  }
}

// Step 3: Configure Project Settings
PATCH /api/projects/507f1f77bcf86cd799439011
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "settings": {
    "auth": {
      "providers": {
        "local": { "enabled": true },
        "google": { "enabled": true },
        "github": { "enabled": true },
        "magicLink": { "enabled": true },
        "otp": { "enabled": true }
      },
      "requireEmailVerification": true,
      "sessionTimeout": 1800000,
      "passwordPolicy": {
        "minLength": 8,
        "requireUppercase": true,
        "requireNumbers": true,
        "requireSpecialChars": true
      }
    },
    "database": {
      "collections": {
        "users": { "enabled": true },
        "products": { "enabled": true },
        "orders": { "enabled": true }
      }
    },
    "storage": {
      "maxFileSize": 10485760, // 10MB
      "allowedTypes": ["image/jpeg", "image/png", "application/pdf"]
    }
  }
}

// Step 4: Set up Payment Gateway (for billing)
POST /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "provider": "paystack",
  "publicKey": "pk_test_abc123...",
  "secretKey": "[SECRET_KEY_NOT_RETURNED]", // Only sent during creation, never returned
  "config": {
    "currency": "NGN",
    "webhookUrl": "https://api.mudbase.com/api/billing/webhooks/paystack"
  }
}

// Response
{
  "message": "Payment gateway created successfully",
  "gateway": {
    "_id": "507f1f77bcf86cd799439014",
    "provider": "paystack",
    "status": "pending"
    // NOTE: secretKey and webhookSecret are NEVER returned in responses
  }
}

// Step 5: Activate Payment Gateway
PATCH /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "status": "active"
}
```

---

## Wallet as a Service Workflows

### Important Security Notes

1. **Private Key Generation**: Each wallet gets its own unique private key. Keys are generated per wallet, not shared per project or organization.
2. **Private Key Visibility**: Private keys are ONLY shown once in the `/api/wallet/generate-key` endpoint response. They are NEVER returned in:
   - Wallet creation responses
   - Wallet listing responses
   - Wallet balance responses
   - Any other wallet-related endpoints
3. **Encryption**: All private keys are encrypted using AES-256-GCM before storage in the database.
4. **Key Storage**: Users must save the private key securely when generated. It cannot be retrieved later.

### Understanding Wallet Encryption

**Two Types of Keys**:

1. **Project Encryption Key** (`walletEncryptionKey`):
   - One per project (auto-generated)
   - Used to encrypt/decrypt wallet private keys for database storage
   - Provides additional security layer and project isolation
   - **NOT used for signing transactions**

2. **Wallet Private Keys**:
   - One unique key per wallet (generated per wallet)
   - The actual cryptocurrency private key
   - Used to sign blockchain transactions
   - Encrypted using project encryption key before storage

**Transaction Flow**:
- When making a transaction, the wallet's private key is decrypted using the project encryption key
- The decrypted wallet private key is then used to sign the transaction
- The project encryption key is NEVER used to sign transactions - only the wallet's private key

For detailed explanation, see `docs/WALLET_ENCRYPTION_EXPLAINED.md`.

### Supported Cryptocurrencies
- **BTC** (Bitcoin) - Bech32 addresses (bc1...)
- **ETH** (Ethereum)
- **BNB** (Binance Smart Chain)
- **SOL** (Solana)
- **TRX** (Tron)
- **LTC** (Litecoin)
- **USDT** (Tether - Multi-network: ETH, TRX, BSC, SOL, POLYGON)

### Platform Fee Structure (Hybrid Model)

**Formula**: `Platform Fee = max(1% * amount, minimumFee)`

| Currency | Minimum Fee | Minimum Withdrawal | Notes |
|----------|-------------|-------------------|-------|
| BTC | 0.00012 BTC (~$8) | 0.0002 BTC (~$13) | Updated based on market rates |
| ETH | 0.001 ETH (~$2.50) | 0.001 ETH (~$2.50) | Increased to protect against gas spikes |
| BNB | 0.0002 BNB (~$0.13) | 0.001 BNB (~$0.65) | Updated based on market rates |
| SOL | 0.008 SOL (~$1.20) | 0.01 SOL (~$1.50) | Platform as feePayer (no pre-fund) |
| TRX | 1 TRX (~$0.05) | 5 TRX (~$0.25) | Lowered for micro-transactions support |
| LTC | 0.0001 LTC (~$0.01) | 0.001 LTC (~$0.10) | Updated based on market rates |
| USDT-ETH | $4 fixed | 5 USDT | Covers gas spikes |
| USDT-BSC | $0.75 fixed | 2 USDT | Scaled with network cost + profit margin |
| USDT-TRX | $0.50 fixed | 2 USDT | Scaled with network cost + profit margin |
| USDT-SOL | $0.15 fixed | 2 USDT | Scaled with network cost + profit margin |

**Profitability Guardrails**: Transactions are rejected if platform fee < estimated network cost + pre-fund cost.

**Important Notes**:
- **ETH Gas Spikes**: ETH minimum fee (0.001 ETH) protects against gas spikes, but network conditions may still cause fees to exceed minimum during extreme congestion. Consider implementing dynamic fee adjustments for ETH during high gas periods.
- **USDT Fees**: Fees on non-ETH chains (BSC, TRX, SOL) are scaled with network cost + profit margin rather than flat rates for better user experience and competitiveness.
- **Micro-Transactions**: TRX minimum withdrawal lowered to 5 TRX to support small users and micro-transactions.

For detailed fee structure and rationale, see [FEE_STRUCTURE_UPDATED.md](./FEE_STRUCTURE_UPDATED.md)

### 1. Generate Private Key (Dashboard Endpoint)

**Important**: Each wallet gets its own unique private key. Keys are generated per wallet, not per project. The private key is only shown once in this response and must be saved securely.

```javascript
// Generate a new key pair for any supported currency
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "network": null // Only required for USDT
}

// Response
{
  "success": true,
  "data": {
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "privateKey": "L5EZftvrYaSudiozVRzTqLcHLNDoVn7H5HSfM9BAN6pMA5zoi3WZ",
    "publicKey": "02a1633cafcc01ebfb6d78e39f687a1f0995c62fc95f51ead10a02ee0be551b5fb"
  },
  "warning": "This private key is shown only once. Store it securely."
}

// For USDT with network
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "USDT",
  "network": "ETH" // Options: ETH, TRX, BSC, SOL, POLYGON
}

// Response
{
  "success": true,
  "data": {
    "currency": "USDT",
    "network": "ETH",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "privateKey": "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
    "publicKey": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
  },
  "warning": "This private key is shown only once. Store it securely."
}
```

### 2. Create Wallet with Generated Private Key

**Security Note**: Private keys are NEVER returned in wallet creation responses. They are encrypted and stored securely. Only the address and wallet metadata are returned.

```javascript
// User can set the generated private key from dashboard
POST /api/wallet/create
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "privateKey": "[PRIVATE_KEY_FROM_GENERATE_KEY_ENDPOINT]",
  "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh"
}

// Response
{
  "success": true,
  "message": "BTC wallet created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439015",
    "user": "507f1f77bcf86cd799439012",
    "org": "507f1f77bcf86cd799439013",
    "project": "507f1f77bcf86cd799439011",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "balance": "0",
    "isCustomKey": true,
    "isActive": true,
    "createdAt": "2024-01-15T10:00:00Z",
    "updatedAt": "2024-01-15T10:00:00Z"
  }
}
```

### 3. Create Wallet with Auto-Generated Key

```javascript
// Let system generate the key pair
POST /api/wallet/create
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "ETH"
}

// Response
{
  "success": true,
  "message": "ETH wallet created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439016",
    "user": "507f1f77bcf86cd799439012",
    "org": "507f1f77bcf86cd799439013",
    "project": "507f1f77bcf86cd799439011",
    "currency": "ETH",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "balance": "0",
    "isCustomKey": false,
    "isActive": true,
    "createdAt": "2024-01-15T10:00:00Z",
    "updatedAt": "2024-01-15T10:00:00Z"
  }
}
```

### 4. Get User Wallets

```javascript
// Get all wallets for authenticated user
GET /api/wallet
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": [
    {
      "_id": "507f1f77bcf86cd799439015",
      "currency": "BTC",
      "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
      "balance": "0.05",
      "isCustomKey": true,
      "isActive": true,
      "project": "507f1f77bcf86cd799439011",
      "createdAt": "2024-01-15T10:00:00Z"
    },
    {
      "_id": "507f1f77bcf86cd799439016",
      "currency": "ETH",
      "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
      "balance": "2.5",
      "isCustomKey": false,
      "isActive": true,
      "project": "507f1f77bcf86cd799439011",
      "createdAt": "2024-01-15T10:00:00Z"
    }
  ]
}
```

### 4.5. Get Wallet Private Key

**Security Warning**: This endpoint returns the decrypted private key. Use only when you need to export your wallet. The private key is sensitive and should be kept secure.

```javascript
// Get private key for a specific wallet
GET /api/wallet/{walletId}/private-key
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "walletId": "507f1f77bcf86cd799439015",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "privateKey": "L5EZftvrYaSudiozVRzTqLcHLNDoVn7H5HSfM9BAN6pMA5zoi3WZ",
    "isCustomKey": true
  },
  "warning": "Keep this private key secure and never share it. Anyone with access to this key can control your wallet.",
  "security": {
    "accessedAt": "2024-01-15T11:20:00Z",
    "accessedBy": "507f1f77bcf86cd799439012"
  }
}
```

**Access Control**:
- User can only retrieve private keys for wallets they own
- Wallet must belong to the user's organization
- All access is logged for security audit

### 6. Get Wallet Balance

```javascript
// Get balance for specific wallet
GET /api/wallet/507f1f77bcf86cd799439015/balance
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "walletId": "507f1f77bcf86cd799439015",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "balance": "0.05",
    "balanceInUSD": 2500.00,
    "lastSyncedAt": "2024-01-15T10:30:00Z"
  }
}
```

### 7. Withdraw Funds (Async Processing)

**Important**: Withdrawals are processed asynchronously. The API returns immediately with a `queued` status. You must check the transaction status to see when it's confirmed. Real blockchain confirmations take time (BTC: 10-60 minutes, ETH: 15-45 seconds, SOL: 5-10 seconds, etc.).

**Fee Model**: Platform fees use a hybrid model: `max(1% * amount, minimumFee)`. Project fees (optional) are added on top if configured. Minimum fees are enforced per currency to ensure profitability.

```javascript
// Withdraw from wallet
POST /api/wallet/507f1f77bcf86cd799439015/withdraw
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "toAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "amount": 0.01,
  "feeRate": 10 // For BTC/LTC only
}

// Response (Returns Immediately - Async Processing)
{
  "success": true,
  "message": "Withdrawal is processing",
  "data": {
    "transactionId": "507f1f77bcf86cd799439027",
    "status": "queued",
    "amount": 0.01,
    "platformFee": 0.00012, // max(1% * 0.01, 0.00012 BTC minimum)
    "projectFee": 0.00005, // Optional: if project fee is configured
    "totalFee": 0.00017, // Platform fee + project fee
    "message": "Withdrawal is processing. Check transaction status for updates."
  }
}

// Check Transaction Status
GET /api/wallet/transactions/507f1f77bcf86cd799439027
Authorization: Bearer {user_token}

// Response (During Processing)
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439027",
    "status": "prefunding", // or "processing", "broadcasted"
    "mainTxStatus": "pending",
    "mainTxHash": null,
    "platformFee": 0.0005,
    "createdAt": "2024-01-15T10:35:00Z"
  }
}

// Response (After Confirmation)
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439027",
    "status": "confirmed",
    "mainTxHash": "abc123def456ghi789jkl012mno345pqr678stu901vwx234yz",
    "mainTxStatus": "confirmed",
    "refundTxHash": "xyz789abc123def456ghi789jkl012mno345pqr678",
    "refundStatus": "confirmed",
    "networkFee": 0.0001,
    "platformFee": 0.0005,
    "completedAt": "2024-01-15T10:45:00Z"
  }
}
```

**Transaction Status Flow**:
- `queued` → Transaction added to processing queue
- `prefunding` → Platform sending pre-fund (if needed)
- `processing` → User transaction being prepared
- `broadcasted` → Transaction broadcast to blockchain
- `confirmed` → Transaction confirmed on blockchain
- `completed` → All operations completed successfully
- `failed` → Transaction failed (check error field)
- `partial` → User tx succeeded but refund failed (CRITICAL - requires manual intervention)

### 8. Validate Address

```javascript
// Validate cryptocurrency address
POST /api/wallet/validate-address
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "currency": "BTC"
}

// Response
{
  "success": true,
  "data": {
    "isValid": true,
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "format": "bech32"
  }
}
```

### 9. Get Supported Currencies

```javascript
// Get list of supported currencies
GET /api/wallet/currencies
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": [
    {
      "code": "BTC",
      "name": "Bitcoin",
      "network": "mainnet"
    },
    {
      "code": "ETH",
      "name": "Ethereum",
      "network": "mainnet"
    },
    {
      "code": "BNB",
      "name": "Binance Coin",
      "network": "bsc"
    },
    {
      "code": "LTC",
      "name": "Litecoin",
      "network": "mainnet"
    },
    {
      "code": "SOL",
      "name": "Solana",
      "network": "mainnet"
    },
    {
      "code": "TRX",
      "name": "Tron",
      "network": "mainnet"
    },
    {
      "code": "USDT",
      "name": "Tether",
      "network": "ethereum"
    }
  ]
}
```

---

## Billing & Payment Gateway Workflows

### 1. Setup Payment Gateway

```javascript
// Create payment gateway account
POST /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "provider": "paystack", // or "flutterwave", "monnify", "interswitch"
  "publicKey": "pk_test_abc123...",
  "secretKey": "[SECRET_KEY_NOT_RETURNED]",
  "config": {
    "currency": "NGN",
    "webhookUrl": "https://api.mudbase.com/api/billing/webhooks/paystack"
  }
}

// Response
{
  "message": "Payment gateway created successfully",
  "gateway": {
    "_id": "507f1f77bcf86cd799439017",
    "provider": "paystack",
    "status": "pending",
    "publicKey": "pk_test_abc123...",
    "createdAt": "2024-01-15T10:40:00Z"
  }
}

// Activate gateway
PATCH /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "status": "active"
}
```

### 2. Create Subscription Plan

```javascript
// Create a plan for your project
POST /api/billing/projects/507f1f77bcf86cd799439011/plans
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Pro Plan",
  "description": "Professional plan with advanced features",
  "pricing": {
    "type": "recurring",
    "monthly": {
      "amount": 5000, // in kobo (50 NGN)
      "currency": "NGN"
    },
    "yearly": {
      "amount": 50000, // in kobo (500 NGN)
      "currency": "NGN"
    },
    "usage": [
      {
        "metric": "api_calls",
        "unitPrice": 0.1, // per API call
        "includedUnits": 10000,
        "currency": "NGN"
      }
    ]
  },
  "features": [
    {
      "name": "api_access",
      "description": "Full API access",
      "included": true
    },
    {
      "name": "storage",
      "description": "100GB storage",
      "included": true,
      "limit": 107374182400 // 100GB in bytes
    }
  ],
  "limits": {
    "apiCalls": 100000,
    "storage": 107374182400,
    "bandwidth": 1073741824000
  },
  "trial": {
    "enabled": true,
    "days": 7
  },
  "isActive": true,
  "isDefault": false
}

// Response
{
  "message": "Plan created successfully",
  "plan": {
    "_id": "507f1f77bcf86cd799439018",
    "name": "Pro Plan",
    "description": "Professional plan with advanced features",
    "pricing": {
      "type": "recurring",
      "monthly": {
        "amount": 5000,
        "currency": "NGN"
      },
      "yearly": {
        "amount": 50000,
        "currency": "NGN"
      }
    },
    "features": [
      {
        "name": "api_access",
        "description": "Full API access",
        "included": true
      },
      {
        "name": "storage",
        "description": "100GB storage",
        "included": true,
        "limit": 107374182400
      }
    ],
    "isActive": true,
    "createdAt": "2024-01-15T10:40:00Z"
  }
}
```

### 3. Customer Checkout Flow

```javascript
// Step 1: Get available plans (Public API)
GET /api/billing/public/projects/507f1f77bcf86cd799439011/plans

// Response
{
  "plans": [
    {
      "_id": "507f1f77bcf86cd799439018",
      "name": "Pro Plan",
      "description": "Professional plan with advanced features",
      "pricing": {
        "monthly": {
          "amount": 5000,
          "currency": "NGN"
        },
        "yearly": {
          "amount": 50000,
          "currency": "NGN"
        }
      },
      "features": [ ... ]
    }
  ]
}

// Step 2: Create checkout session
POST /api/billing/public/projects/507f1f77bcf86cd799439011/checkout
Content-Type: application/json

{
  "planId": "507f1f77bcf86cd799439018",
  "billingCycle": "monthly",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  },
  "successUrl": "https://yourapp.com/billing/success",
  "cancelUrl": "https://yourapp.com/billing/cancel"
}

// Response
{
  "success": true,
  "data": {
    "checkoutUrl": "https://paystack.com/pay/abc123...",
    "reference": "mudbase_1705315200_abc123def456",
    "amount": 5000,
    "currency": "NGN"
  }
}

// Step 3: Redirect customer to authorizationUrl
// Customer completes payment on Paystack/Flutterwave

// Step 4: Payment Gateway redirects to callback URL
// Backend automatically verifies payment and creates subscription

// Step 5: Verify payment (if needed manually)
POST /api/billing/public/projects/507f1f77bcf86cd799439011/verify-payment
Content-Type: application/json

{
  "reference": "mudbase_1705315200_abc123def456",
  "orgId": "507f1f77bcf86cd799439013"
}

// Response
{
  "success": true,
  "message": "Payment verified and subscription created",
  "data": {
    "subscription": {
      "_id": "507f1f77bcf86cd799439019",
      "status": "active",
      "plan": {
        "_id": "507f1f77bcf86cd799439018",
        "name": "Pro Plan"
      },
      "customerEmail": "customer@example.com",
      "currentPeriodEnd": "2024-02-15T10:45:00Z",
      "billingCycle": "monthly"
    }
  }
}
```

### 4. Check Subscription Status

```javascript
// Check customer subscription (Public API)
GET /api/billing/public/projects/507f1f77bcf86cd799439011/subscription?email=customer@example.com

// Response
{
  "hasSubscription": true,
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active",
    "plan": {
      "_id": "507f1f77bcf86cd799439018",
      "name": "Pro Plan",
      "price": 5000,
      "currency": "NGN"
    },
    "customerEmail": "customer@example.com",
    "currentPeriodEnd": "2024-02-15T10:45:00Z",
    "billingCycle": "monthly",
    "createdAt": "2024-01-15T10:45:00Z"
  }
}
```

### 5. Check Feature Access

```javascript
// Check if customer has access to a feature
GET /api/billing/public/projects/507f1f77bcf86cd799439011/feature-access?email=customer@example.com&feature=api_access

// Response
{
  "hasAccess": true,
  "reason": "Active subscription",
  "plan": {
    "_id": "507f1f77bcf86cd799439018",
    "name": "Pro Plan"
  },
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active"
  }
}
```

### 6. Record Usage (Metered Billing)

```javascript
// Record usage for metered billing
POST /api/billing/public/projects/507f1f77bcf86cd799439011/usage
Content-Type: application/json

{
  "email": "customer@example.com",
  "metric": "api_calls",
  "quantity": 150
}

// Response
{
  "success": true,
  "message": "Usage recorded successfully"
}
```

### 7. Cancel Subscription

```javascript
// Cancel subscription
POST /api/billing/subscriptions/507f1f77bcf86cd799439019/cancel
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "cancelImmediately": false // Cancel at period end
}

// Response
{
  "message": "Subscription canceled successfully",
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active",
    "cancelAtPeriodEnd": true,
    "currentPeriodEnd": "2024-02-15T10:45:00Z",
    "canceledAt": "2024-01-15T11:00:00Z"
  }
}
```

### 8. Payment Gateway Webhook

```javascript
// Payment gateway sends webhook on payment events
POST /api/billing/webhooks/paystack
Content-Type: application/json
X-Paystack-Signature: {signature}

{
  "event": "charge.success",
  "data": {
    "reference": "mudbase_1705315200_abc123def456",
    "amount": 5000,
    "currency": "NGN",
    "customer": {
      "email": "customer@example.com"
    },
    "metadata": {
      "projectId": "507f1f77bcf86cd799439011",
      "planId": "507f1f77bcf86cd799439018",
      "billingCycle": "monthly",
      "orgId": "507f1f77bcf86cd799439013"
    }
  }
}

// Backend automatically:
// 1. Verifies webhook signature
// 2. Verifies payment
// 3. Creates subscription
// 4. Sends confirmation email
// 5. Triggers project webhook
```

---

## Database Operations

### 1. Create Collection Schema

```javascript
// Define collection schema
POST /api/projects/507f1f77bcf86cd799439011/schemas
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "products",
  "fields": [
    {
      "name": "name",
      "type": "string",
      "required": true,
      "indexed": true
    },
    {
      "name": "price",
      "type": "number",
      "required": true
    },
    {
      "name": "description",
      "type": "string"
    },
    {
      "name": "category",
      "type": "string",
      "indexed": true
    },
    {
      "name": "inStock",
      "type": "boolean",
      "default": true
    },
    {
      "name": "createdAt",
      "type": "date",
      "default": "now"
    }
  ],
  "indexes": [
    {
      "fields": ["category", "price"],
      "unique": false
    }
  ]
}

// Response
{
  "success": true,
  "collection": {
    "_id": "507f1f77bcf86cd799439020",
    "name": "products",
    "slug": "products",
    "project": "507f1f77bcf86cd799439011",
    "fields": [
      {
        "name": "name",
        "type": "string",
        "required": true,
        "indexed": true
      },
      {
        "name": "price",
        "type": "number",
        "required": true
      },
      {
        "name": "description",
        "type": "string"
      },
      {
        "name": "category",
        "type": "string",
        "indexed": true
      },
      {
        "name": "inStock",
        "type": "boolean",
        "default": true
      }
    ],
    "createdAt": "2024-01-15T10:50:00Z"
  }
}
```

### 2. Create Document

```javascript
// Create a document in collection
POST /api/projects/507f1f77bcf86cd799439011/data/products
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Laptop",
  "price": 999.99,
  "description": "High-performance laptop",
  "category": "electronics",
  "inStock": true
}

// Response
{
  "message": "Data created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439021",
    "name": "Laptop",
    "price": 999.99,
    "description": "High-performance laptop",
    "category": "electronics",
    "inStock": true,
    "createdAt": "2024-01-15T10:55:00Z",
    "updatedAt": "2024-01-15T10:55:00Z"
  }
}
```

### 3. Query Documents

```javascript
// Get all documents with filters
GET /api/projects/507f1f77bcf86cd799439011/data/products?category=electronics&price[gte]=500&limit=10&page=1
Authorization: Bearer {user_token}

// Response
{
  "data": [
    {
      "_id": "507f1f77bcf86cd799439021",
      "name": "Laptop",
      "price": 999.99,
      "description": "High-performance laptop",
      "category": "electronics",
      "inStock": true,
      "createdAt": "2024-01-15T10:55:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 1,
    "totalPages": 1
  }
}

// Advanced query
POST /api/projects/507f1f77bcf86cd799439011/data/products/query
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "filter": {
    "$and": [
      { "category": "electronics" },
      { "price": { "$gte": 500 } },
      { "inStock": true }
    ]
  },
  "sort": { "price": -1 },
  "limit": 10,
  "skip": 0
}
```

### 4. Update Document

```javascript
// Update document
PATCH /api/projects/507f1f77bcf86cd799439011/data/products/507f1f77bcf86cd799439021
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "price": 899.99,
  "inStock": false
}

// Response
{
  "message": "Data updated successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439021",
    "name": "Laptop",
    "price": 899.99,
    "description": "High-performance laptop",
    "category": "electronics",
    "inStock": false,
    "createdAt": "2024-01-15T10:55:00Z",
    "updatedAt": "2024-01-15T11:00:00Z"
  }
}
```

### 5. Delete Document

```javascript
// Delete document
DELETE /api/projects/507f1f77bcf86cd799439011/data/products/507f1f77bcf86cd799439021
Authorization: Bearer {user_token}

// Response
{
  "message": "Data deleted successfully"
}
```

---

## Real-time Chat System

### 1. Create Chat

```javascript
// Create a new chat
POST /api/projects/507f1f77bcf86cd799439011/chats
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Team Discussion",
  "type": "group", // or "direct"
  "participants": [
    "507f1f77bcf86cd799439012",
    "507f1f77bcf86cd799439022"
  ],
  "metadata": {
    "projectId": "507f1f77bcf86cd799439011"
  }
}

// Response
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439023",
    "name": "Team Discussion",
    "type": "group",
    "participants": [
      {
        "user": "507f1f77bcf86cd799439012",
        "role": "admin",
        "joinedAt": "2024-01-15T11:05:00Z"
      },
      {
        "user": "507f1f77bcf86cd799439022",
        "role": "member",
        "joinedAt": "2024-01-15T11:05:00Z"
      }
    ],
    "createdBy": "507f1f77bcf86cd799439012",
    "project": "507f1f77bcf86cd799439011",
    "createdAt": "2024-01-15T11:05:00Z"
  }
}
```

### 2. Send Message (HTTP)

```javascript
// Send message via HTTP
POST /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "content": "Hello team! How's everyone doing?",
  "type": "text" // or "image", "file", etc.
}

// Response
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439024",
    "content": "Hello team! How's everyone doing?",
    "type": "text",
    "sender": "507f1f77bcf86cd799439012",
    "chat": "507f1f77bcf86cd799439023",
    "project": "507f1f77bcf86cd799439011",
    "createdAt": "2024-01-15T11:06:00Z",
    "updatedAt": "2024-01-15T11:06:00Z"
  }
}
```

### 3. Real-time Messaging (Socket.IO)

```javascript
// Client-side Socket.IO connection
import io from 'socket.io-client';

const socket = io('https://api.mudbase.com', {
  auth: {
    token: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
  }
});

// Join chat room
socket.emit('chat:join', {
  chatId: '507f1f77bcf86cd799439023',
  projectId: '507f1f77bcf86cd799439011'
});

// Send message via Socket.IO
socket.emit('chat:message:send', {
  chatId: '507f1f77bcf86cd799439023',
  content: 'Hello from real-time!',
  type: 'text'
});

// Listen for new messages
socket.on('chat:message:new', (data) => {
  console.log('New message:', data);
  // {
  //   message: {
  //     _id: '507f1f77bcf86cd799439025',
  //     content: 'Hello from real-time!',
  //     sender: { ... },
  //     createdAt: '2024-01-15T11:07:00Z'
  //   },
  //   chatId: '507f1f77bcf86cd799439023'
  // }
});

// Typing indicator
socket.emit('chat:typing', {
  chatId: '507f1f77bcf86cd799439023',
  isTyping: true
});

socket.on('chat:typing', (data) => {
  console.log('User typing:', data);
  // {
  //   userId: '507f1f77bcf86cd799439012',
  //   chatId: '507f1f77bcf86cd799439023',
  //   isTyping: true
  // }
});

// Voice/Video call events
socket.emit('chat:call:initiate', {
  chatId: '507f1f77bcf86cd799439023',
  type: 'video' // or 'voice'
});

socket.on('chat:call:incoming', (data) => {
  console.log('Incoming call:', data);
});

socket.emit('chat:call:accept', {
  callId: 'call_abc123'
});

socket.emit('chat:call:reject', {
  callId: 'call_abc123'
});

socket.emit('chat:call:end', {
  callId: 'call_abc123'
});
```

### 4. Edit Message

```javascript
// Edit message
PATCH /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "content": "Hello team! How's everyone doing? (edited)"
}

// Socket.IO event also emitted: 'chat:message:updated'
```

### 5. Delete Message

```javascript
// Delete message
DELETE /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024
Authorization: Bearer {user_token}

// Socket.IO event also emitted: 'chat:message:deleted'
```

### 6. Add Reaction

```javascript
// Add reaction to message
POST /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024/reactions
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "emoji": "👍"
}

// Socket.IO event also emitted: 'chat:message:reaction'
```

---

## Integration System

### 1. Create Integration

```javascript
// Create integration
POST /api/projects/507f1f77bcf86cd799439011/integrations
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Twitter API Integration",
  "provider": "twitter",
  "category": "social",
    "settings": {
      "isActive": true
      // NOTE: API keys, secrets, and tokens are encrypted and never returned in responses
    },
  "config": {
    "rateLimit": 100,
    "timeout": 5000
  }
}

// Response
{
  "integration": {
    "_id": "507f1f77bcf86cd799439026",
    "name": "Twitter API Integration",
    "provider": "twitter",
    "category": "social",
    "project": "507f1f77bcf86cd799439011",
    "settings": {
      "isActive": true
    },
    "createdAt": "2024-01-15T11:10:00Z",
    "updatedAt": "2024-01-15T11:10:00Z"
  }
}
```

### 2. Execute Integration

```javascript
// Execute integration
POST /api/projects/507f1f77bcf86cd799439011/integrations/507f1f77bcf86cd799439026/execute
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "method": "POST",
  "endpoint": "/2/tweets",
  "body": {
    "text": "Hello from MUDBASE!"
  },
  "headers": {
    "Content-Type": "application/json"
  }
}

// Response
{
  "success": true,
  "data": {
    "status": 201,
    "data": {
      "id": "1234567890",
      "text": "Hello from MUDBASE!",
      "created_at": "2024-01-15T11:12:00Z"
    },
    "headers": {
      "content-type": "application/json",
      "x-rate-limit-remaining": "299"
    }
  },
  "usage": {
    "apiCalls": 1,
    "timestamp": "2024-01-15T11:12:00Z"
  }
}
```

### 3. Get Integration Usage Stats

```javascript
// Get usage statistics
GET /api/projects/507f1f77bcf86cd799439011/integrations/507f1f77bcf86cd799439026/usage?period=month
Authorization: Bearer {user_token}

// Response
{
  "stats": {
    "totalCalls": 1250,
    "successfulCalls": 1200,
    "failedCalls": 50,
    "successRate": 96.0,
    "period": "month",
    "breakdown": [
      {
        "date": "2024-01-15",
        "calls": 45,
        "successful": 43,
        "failed": 2
      }
    ]
  }
}
```

---

## Complete End-to-End Scenarios

### Scenario 1: E-commerce Platform Setup

```javascript
// Step 1: Register Admin User
POST /api/auth/local/register
{
  "email": "admin@ecommerce.com",
  "password": "SecurePass123!",
  "firstName": "Admin",
  "lastName": "User"
}

// Step 2: Create Organization
POST /api/orgs
Authorization: Bearer {admin_token}
{
  "name": "Ecommerce Corp"
}

// Step 3: Create Project
POST /api/projects
Authorization: Bearer {admin_token}
{
  "name": "Ecommerce Platform",
  "orgId": "{org_id}"
}

// Step 4: Setup Payment Gateway
POST /api/billing/orgs/{org_id}/payment-gateway
Authorization: Bearer {admin_token}
{
  "provider": "paystack",
  "publicKey": "pk_test_...",
  "secretKey": "sk_test_..."
}

// Step 5: Create Product Schema
POST /api/projects/{project_id}/schemas
Authorization: Bearer {admin_token}
{
  "name": "products",
  "fields": [
    { "name": "name", "type": "string", "required": true },
    { "name": "price", "type": "number", "required": true },
    { "name": "stock", "type": "number", "required": true }
  ]
}

// Step 6: Create Order Schema
POST /api/projects/{project_id}/schemas
Authorization: Bearer {admin_token}
{
  "name": "orders",
  "fields": [
    { "name": "customerId", "type": "string", "required": true },
    { "name": "products", "type": "array", "required": true },
    { "name": "total", "type": "number", "required": true },
    { "name": "status", "type": "string", "default": "pending" }
  ]
}

// Step 7: Customer Registration
POST /api/auth/local/register
{
  "email": "customer@example.com",
  "password": "CustomerPass123!",
  "firstName": "John",
  "lastName": "Customer",
  "projectId": "{project_id}"
}

// Step 8: Customer Creates Order
POST /api/projects/{project_id}/data/orders
Authorization: Bearer {customer_token}
{
  "customerId": "{customer_id}",
  "products": [
    { "productId": "{product_id}", "quantity": 2 }
  ],
  "total": 1999.98,
  "status": "pending"
}

// Step 9: Process Payment
POST /api/billing/public/projects/{project_id}/checkout
{
  "planId": "{plan_id}",
  "billingCycle": "one-time",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  },
  "amount": 1999.98
}

// Step 10: Update Order Status
PATCH /api/projects/{project_id}/data/orders/{order_id}
Authorization: Bearer {admin_token}
{
  "status": "paid"
}
```

### Scenario 2: Crypto Wallet Integration

```javascript
// Step 1: User generates Bitcoin wallet key
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
{
  "currency": "BTC"
}

// Response includes privateKey (shown once only) that user must save securely

// Step 2: User creates wallet with generated key
POST /api/wallet/create
Authorization: Bearer {user_token}
{
  "currency": "BTC",
  "privateKey": "[PRIVATE_KEY_FROM_STEP_1]",
  "address": "[ADDRESS_FROM_STEP_1]"
}

// Response does NOT include privateKey - it's encrypted and stored securely

// Step 3: Check wallet balance
GET /api/wallet/{wallet_id}/balance
Authorization: Bearer {user_token}

// Step 4: Receive payment (external)
// Bitcoin sent to wallet address

// Step 5: Withdraw funds (async processing)
POST /api/wallet/{wallet_id}/withdraw
Authorization: Bearer {user_token}
{
  "toAddress": "bc1q...",
  "amount": 0.01,
  "feeRate": 10
}

// Response (returns immediately - async processing)
{
  "success": true,
  "data": {
    "transactionId": "...",
    "status": "queued",
    "platformFee": 0.0005 // max(1% * 0.01, 0.0005 BTC minimum)
  }
}

// Step 6: Check transaction status (poll until confirmed)
GET /api/wallet/transactions/{transaction_id}
Authorization: Bearer {user_token}

// Response (after confirmation - may take 10-60 minutes for BTC)
{
  "success": true,
  "data": {
    "status": "confirmed",
    "mainTxHash": "...",
    "mainTxStatus": "confirmed"
  }
}
```

### Scenario 3: SaaS Subscription Flow

```javascript
// Step 1: Setup billing plan
POST /api/billing/projects/{project_id}/plans
Authorization: Bearer {admin_token}
{
  "name": "Premium",
  "pricing": {
    "monthly": { "amount": 10000, "currency": "NGN" },
    "yearly": { "amount": 100000, "currency": "NGN" }
  },
  "features": [
    { "name": "api_access", "included": true },
    { "name": "storage", "included": true, "limit": 107374182400 }
  ]
}

// Step 2: Customer subscribes
POST /api/billing/public/projects/{project_id}/checkout
{
  "planId": "{plan_id}",
  "billingCycle": "monthly",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  }
}

// Step 3: Check subscription status
GET /api/billing/public/projects/{project_id}/subscription?email=customer@example.com

// Step 4: Check feature access
GET /api/billing/public/projects/{project_id}/feature-access?email=customer@example.com&feature=api_access

// Step 5: Record usage
POST /api/billing/public/projects/{project_id}/usage
{
  "email": "customer@example.com",
  "metric": "api_calls",
  "quantity": 150
}
```

---

## API Reference Quick Guide

### Authentication Endpoints

```
POST   /api/auth/register                - Register new user (org-based)
POST   /api/auth/login                   - Login user (org-based)
POST   /api/auth/logout                  - Logout user (org-based)
GET    /api/auth/session                 - Get current session (org-based)
POST   /api/auth/password-reset          - Request password reset (org-based)
POST   /api/auth/password-reset/{token}  - Reset password (org-based)
POST   /api/auth/local/register          - Register new user (project-based)
POST   /api/auth/local/login             - Login user (project-based)
POST   /api/auth/local/logout            - Logout user (project-based)
GET    /api/auth/local/session           - Get current session (project-based)
POST   /api/auth/local/password-reset    - Request password reset (project-based)
POST   /api/auth/local/password-reset/{token} - Reset password (project-based)
GET    /api/auth/oauth/{provider}        - Initiate OAuth
GET    /api/auth/oauth/{provider}/callback - OAuth callback
POST   /api/auth/magic-link/send         - Send magic link
GET    /api/auth/magic-link/verify        - Verify magic link
POST   /api/auth/otp/send                - Send OTP
POST   /api/auth/otp/verify              - Verify OTP
POST   /api/auth/2fa/setup               - Setup 2FA
POST   /api/auth/2fa/verify              - Verify 2FA
```

### Wallet Endpoints

```
GET    /api/wallet/currencies            - Get supported currencies
POST   /api/wallet/generate-key          - Generate key pair
POST   /api/wallet/validate-address      - Validate address
POST   /api/wallet/create                 - Create wallet
GET    /api/wallet                        - Get user wallets
GET    /api/wallet/{id}/balance          - Get wallet balance
GET    /api/wallet/{id}/private-key      - Get wallet private key (WARNING: Sensitive)
POST   /api/wallet/{id}/withdraw          - Withdraw funds
```

### Billing Endpoints

```
POST   /api/billing/orgs/{orgId}/payment-gateway - Create gateway
GET    /api/billing/orgs/{orgId}/payment-gateway - Get gateway status
POST   /api/billing/projects/{id}/plans         - Create plan
GET    /api/billing/projects/{id}/plans         - Get plans
GET    /api/billing/public/projects/{id}/plans  - Get public plans
POST   /api/billing/public/projects/{id}/checkout - Create checkout
POST   /api/billing/public/projects/{id}/verify-payment - Verify payment
GET    /api/billing/public/projects/{id}/subscription - Check subscription
GET    /api/billing/public/projects/{id}/feature-access - Check feature access
POST   /api/billing/public/projects/{id}/usage   - Record usage
POST   /api/billing/subscriptions/{id}/cancel   - Cancel subscription
POST   /api/billing/webhooks/{provider}         - Payment webhook
```

### Database Endpoints

```
POST   /api/projects/{id}/schemas         - Create schema
GET    /api/projects/{id}/schemas         - Get schemas
POST   /api/projects/{id}/data/{collection} - Create document
GET    /api/projects/{id}/data/{collection} - Query documents
PATCH  /api/projects/{id}/data/{collection}/{id} - Update document
DELETE /api/projects/{id}/data/{collection}/{id} - Delete document
POST   /api/projects/{id}/data/{collection}/query - Advanced query
```

### Chat Endpoints

```
POST   /api/projects/{id}/chats            - Create chat
GET    /api/projects/{id}/chats           - Get user chats
POST   /api/projects/{id}/chats/{id}/messages - Send message
GET    /api/projects/{id}/chats/{id}/messages - Get messages
PATCH  /api/projects/{id}/chats/{id}/messages/{id} - Edit message
DELETE /api/projects/{id}/chats/{id}/messages/{id} - Delete message
POST   /api/projects/{id}/chats/{id}/messages/{id}/reactions - Add reaction
```

### Integration Endpoints

```
GET    /api/projects/{id}/integrations/templates - Get templates
POST   /api/projects/{id}/integrations          - Create integration
GET    /api/projects/{id}/integrations          - Get integrations
POST   /api/projects/{id}/integrations/{id}/test - Test integration
POST   /api/projects/{id}/integrations/{id}/execute - Execute integration
GET    /api/projects/{id}/integrations/{id}/usage - Get usage stats
```

---

## Security Features

### 1. Session Management
- **Idle Timeout**: 30 minutes of inactivity
- **Rolling Sessions**: Session extends on activity
- **Secure Cookies**: HttpOnly, Secure in production

### 2. Access Control
- **RBAC**: Role-Based Access Control (owner, admin, developer, viewer)
- **Project-Level Access**: Users can only access their project's resources
- **Organization-Level Access**: Users can only access their org's resources

### 3. Encryption
- **Private Keys**: AES-256-GCM encryption at rest
- **API Credentials**: Encrypted in database
- **Password Hashing**: bcrypt with salt rounds 12

### 4. Rate Limiting
- **Global**: 100 requests per 15 minutes per IP
- **Authentication**: 5 login attempts per 15 minutes
- **API Keys**: Configurable per project

### 5. Audit Logging
- All authentication events logged
- All authorization failures logged
- All sensitive operations logged

---

## Best Practices

### 1. Authentication
- Always use HTTPS in production
- Store JWT tokens securely (httpOnly cookies recommended)
- Implement token refresh mechanism
- Use 2FA for sensitive accounts

### 2. Wallet Management
- Never log private keys
- Always validate addresses before transactions
- Use custom keys only when necessary
- Keep backup of private keys securely
- **Async Processing**: Withdrawals are processed asynchronously - check transaction status for updates
- **Real Confirmations**: System uses real blockchain confirmations (not fake delays)
- **Minimum Fees**: Enforced per currency to ensure profitability
- **Status Tracking**: Monitor transaction status through: `queued` → `prefunding` → `processing` → `broadcasted` → `confirmed` → `completed`

### 3. Billing
- Verify webhook signatures
- Handle payment failures gracefully
- Implement retry logic for failed payments
- Monitor subscription status

### 4. Database
- Use indexes for frequently queried fields
- Implement pagination for large datasets
- Validate input data
- Use transactions for critical operations

### 5. Real-time
- Handle connection failures
- Implement reconnection logic
- Use rooms for efficient message delivery
- Clean up on disconnect

---

## Error Handling

### Common Error Codes

```javascript
// Authentication Errors
401 - Unauthorized (Invalid token)
403 - Forbidden (Insufficient permissions)
429 - Too Many Requests (Rate limit exceeded)

// Validation Errors
400 - Bad Request (Invalid input)
422 - Unprocessable Entity (Validation failed)

// Resource Errors
404 - Not Found (Resource doesn't exist)
409 - Conflict (Resource already exists)

// Server Errors
500 - Internal Server Error
503 - Service Unavailable
```

### Error Response Format

```javascript
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "email": "Email is required",
      "password": "Password must be at least 8 characters"
    }
  }
}
```

---

## Project Fee Management

### Overview

Project owners can configure their own fees per currency, which are collected in addition to platform fees. These fees are accumulated and paid out via automated bi-weekly payouts.

### 1. Configure Project Fee Settings

```javascript
// Step 1: Create or update fee settings
POST /api/projects/507f1f77bcf86cd799439011/fee-settings
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "enabled": true,
  "feeAmount": 0.00005, // Must be < platform minimum fee (0.00012 BTC)
  "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "payoutThreshold": 0.001 // Minimum amount before payout
}

// Response
{
  "success": true,
  "message": "Fee settings updated successfully",
  "data": {
    "feeSettings": {
      "BTC": {
        "enabled": true,
        "feeAmount": 0.00005,
        "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "payoutThreshold": 0.001,
        "addressVerified": false
      }
    }
  }
}
```

### 2. Verify Payout Address

```javascript
// Step 1: Initiate address verification
POST /api/projects/507f1f77bcf86cd799439011/fee-settings/BTC/verify-address
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "message": "Test transaction sent. Please confirm receipt or provide transaction hash from the address.",
  "data": {
    "verificationStatus": "pending",
    "testTxHash": "abc123def456...",
    "testAmount": 0.00001,
    "instructions": "Either confirm you received the test transaction, or send a transaction FROM the payout address to prove ownership."
  }
}

// Step 2: Confirm verification
POST /api/projects/507f1f77bcf86cd799439011/fee-settings/BTC/confirm-verification
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "txHash": "xyz789abc123..." // Transaction hash proving ownership
}

// Response
{
  "success": true,
  "message": "Address verified successfully",
  "data": {
    "verified": true,
    "verifiedAt": "2024-01-15T12:00:00Z"
  }
}
```

### 3. Check Fee Balance

```javascript
// Get balance for specific currency
GET /api/projects/507f1f77bcf86cd799439011/fee-balances/BTC
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "currency": "BTC",
    "collectedAmount": 0.0005,
    "threshold": 0.001,
    "status": "accumulating",
    "nextScheduledPayoutDate": null,
    "lastPayoutDate": null,
    "totalPaidOut": 0,
    "totalCollected": 0.0005
  }
}

// Get all balances
GET /api/projects/507f1f77bcf86cd799439011/fee-balances
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "balances": [
      {
        "currency": "BTC",
        "collectedAmount": 0.0005,
        "threshold": 0.001,
        "status": "accumulating"
      },
      {
        "currency": "ETH",
        "collectedAmount": 0.002,
        "threshold": 0.001,
        "status": "ready",
        "nextScheduledPayoutDate": "2024-01-17T02:00:00Z"
      }
    ]
  }
}
```

### 4. View Payout History

```javascript
// Get payout history
GET /api/projects/507f1f77bcf86cd799439011/payout-history?limit=10&offset=0
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "payouts": [
      {
        "_id": "507f1f77bcf86cd799439030",
        "currency": "BTC",
        "grossAmount": 0.001,
        "networkFee": 0.0001,
        "netAmount": 0.0009,
        "toAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "txHash": "abc123def456...",
        "status": "completed",
        "scheduledAt": "2024-01-15T02:00:00Z",
        "processedAt": "2024-01-15T02:05:00Z",
        "confirmedAt": "2024-01-15T02:45:00Z",
        "createdAt": "2024-01-15T02:00:00Z"
      }
    ],
    "total": 1,
    "limit": 10,
    "offset": 0
  }
}
```

### 5. Request Manual Payout

```javascript
// Request manual payout (restricted: once per 30 days per currency)
POST /api/projects/507f1f77bcf86cd799439011/payouts/request-manual
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC"
}

// Response
{
  "success": true,
  "message": "Manual payout requested and processed",
  "data": {
    "payout": {
      "_id": "507f1f77bcf86cd799439031",
      "currency": "BTC",
      "netAmount": 0.0009,
      "txHash": "abc123def456...",
      "status": "completed"
    }
  }
}
```

### 6. Fee Dashboard

```javascript
// Get comprehensive fee dashboard
GET /api/projects/507f1f77bcf86cd799439011/fee-dashboard
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "feeSettings": {
      "BTC": {
        "enabled": true,
        "feeAmount": 0.00005,
        "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "addressVerified": true
      }
    },
    "balances": [
      {
        "currency": "BTC",
        "collectedAmount": 0.0005,
        "threshold": 0.001,
        "status": "accumulating",
        "totalPaidOut": 0.0009,
        "totalCollected": 0.0014
      }
    ],
    "recentPayouts": [
      {
        "_id": "507f1f77bcf86cd799439030",
        "currency": "BTC",
        "netAmount": 0.0009,
        "status": "completed",
        "createdAt": "2024-01-15T02:00:00Z"
      }
    ],
    "totalEarned": 0.0009
  }
}
```

### Important Notes

1. **Fee Validation**: Project fee must be less than platform minimum fee (cannot undercut platform)
2. **Address Verification**: Required before fees are collected (prevents fraud)
3. **Automated Payouts**: Bi-weekly schedule (Wed/Sat 2 AM UTC) when threshold is met
4. **Network Fees**: Deducted from payout amount (platform pays network fees)
5. **Manual Payouts**: Restricted to once per 30 days per currency, requires 25% of threshold
6. **Fee Collection**: Automatically collected during user withdrawals if enabled and verified

---

## Conclusion

This document provides a comprehensive overview of the MUDBASE Backend-as-a-Service platform, including:

- **Multiple Authentication Methods**: Local, OAuth, Magic Link, OTP, 2FA
- **Wallet as a Service**: Support for 7 cryptocurrencies with custom key support
- **Project Fee System**: Project owners can set their own fees with automated bi-weekly payouts
- **Billing System**: Nigerian payment gateways (Paystack, Flutterwave) with subscription management
- **Real-time Features**: Socket.IO-based chat system
- **Database Operations**: Flexible schema-based collections
- **Integration System**: 50+ API integrations
- **Security**: RBAC, encryption, rate limiting, audit logging

For more detailed API documentation, refer to the OpenAPI specification at `/api-docs`.


## Table of Contents
1. [System Architecture Overview](#system-architecture-overview)
2. [Authentication Flows](#authentication-flows)
3. [Organization & Project Setup](#organization--project-setup)
4. [Wallet as a Service Workflows](#wallet-as-a-service-workflows)
5. [Billing & Payment Gateway Workflows](#billing--payment-gateway-workflows)
6. [Database Operations](#database-operations)
7. [Real-time Chat System](#real-time-chat-system)
8. [Integration System](#integration-system)
9. [Complete End-to-End Scenarios](#complete-end-to-end-scenarios)
10. [API Reference Quick Guide](#api-reference-quick-guide)

---

## System Architecture Overview

### Core Components

```
┌────────────────────────────────────────────────────────────┐
│                    MUDBASE Backend Platform                │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Auth Layer  │  │  API Layer   │  │  Socket.IO   │      │
│  │              │  │              │  │  Real-time   │      │
│  │ • Local      │  │ • REST API   │  │ • Chat       │      │
│  │ • OAuth      │  │ • Webhooks   │  │ • Events     │      │
│  │ • Magic Link │  │ • GraphQL    │  │ • Database   │      │
│  │ • OTP        │  │              │  │              │      │
│  │ • 2FA        │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Database   │  │   Storage    │  │  Integrations│      │
│  │              │  │              │  │              │      │
│  │ • MongoDB    │  │ • File Store │  │ • 50+ APIs   │      │
│  │ • Collections│  │ • Buckets    │  │ • Webhooks   │      │
│  │ • Indexes    │  │ • CDN        │  │ • Custom     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Wallet     │  │   Billing    │  │  Security    │      │
│  │   Service    │  │              │  │              │      │
│  │              │  │ • Paystack   │  │ • RBAC       │      │
│  │ • BTC        │  │ • Flutterwave│  │ • Encryption │      │
│  │ • ETH        │  │ • Plans      │  │ • Rate Limit│       │
│  │ • SOL        │  │ • Usage      │  │ • Audit Log  │      │
│  │ • TRX        │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└────────────────────────────────────────────────────────────┘
```

### Data Flow

```
User Request → Authentication → Authorization (RBAC) → Business Logic → Database/External API → Response
                ↓
            JWT Token
                ↓
        Session Management (30min idle timeout)
                ↓
        Real-time Updates via Socket.IO
```

---

## Authentication Flows

### 1. Basic Authentication (Organization-based)

These endpoints are for organization-level authentication and create organizations automatically.

#### Registration Flow

```javascript
// Step 1: User Registration (creates organization automatically)
POST /api/auth/register
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "firstName": "John",
  "lastName": "Doe",
  "orgName": "Acme Corp" // Optional
}

// Response
{
  "message": "User registered successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner",
    "emailVerified": false,
    "org": "507f1f77bcf86cd799439013"
  }
}

// Step 2: Login
POST /api/auth/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]"
}

// Response
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  }
}

// Step 3: Get Session
GET /api/auth/session
Authorization: Bearer {token}

// Response
{
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner"
  },
  "authenticated": true
}

// Step 4: Password Reset Request
POST /api/auth/password-reset
Content-Type: application/json

{
  "email": "john.doe@example.com"
}

// Response
{
  "message": "Password reset email sent"
}

// Step 5: Reset Password
POST /api/auth/password-reset/{token}
Content-Type: application/json

{
  "password": "[NEW_PASSWORD]"
}

// Response
{
  "message": "Password reset successful"
}
```

### 2. Local Authentication (Project-based)

These endpoints are for project-level authentication and require a projectId.

#### Registration Flow

```javascript
// Step 1: User Registration
POST /api/auth/local/register
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "firstName": "John",
  "lastName": "Doe",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "message": "User registered successfully",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "emailVerified": false,
    "role": "developer",
    "org": "507f1f77bcf86cd799439013"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

// Step 2: Email Verification (Optional)
POST /api/auth/verify-email
Authorization: Bearer {token}
Content-Type: application/json

{
  "token": "email-verification-token-from-email"
}

// Response
{
  "success": true,
  "message": "Email verified successfully"
}

// Step 3: Login
POST /api/auth/local/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  },
  "expiresIn": "24h"
}
```

#### Password Reset (Project-based)

```javascript
// Step 1: Request password reset
POST /api/auth/local/password-reset
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "projectId": "507f1f77bcf86cd799439011" // Optional
}

// Response
{
  "message": "Password reset email sent"
}

// Step 2: Reset password
POST /api/auth/local/password-reset/{token}
Content-Type: application/json

{
  "password": "[NEW_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011" // Optional
}

// Response
{
  "message": "Password reset successful"
}
```

#### Session Management

```javascript
// Session expires after 30 minutes of inactivity
// Server automatically extends session on activity (rolling: true)

// Check current session (organization-based)
GET /api/auth/session
Authorization: Bearer {token}

// Check current session (project-based)
GET /api/auth/local/session?projectId={projectId}
Authorization: Bearer {token}

// Response
{
  "user": {
    "id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "project": {
      "id": "507f1f77bcf86cd799439011",
      "name": "My Awesome App",
      "role": "developer"
    }
  },
  "authenticated": true
}

// Response
{
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  }
}

// Refresh token (if implemented)
POST /api/auth/refresh
Authorization: Bearer {token}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": "24h"
}

// Logout (organization-based)
POST /api/auth/logout
Authorization: Bearer {token}

// Response
{
  "message": "Logout successful"
}

// Logout (project-based)
POST /api/auth/local/logout
Authorization: Bearer {token}

// Response
{
  "message": "Logout successful"
}
```

**Key Differences Between Organization-based and Project-based Auth:**

- **Organization-based** (`/api/auth/*`): Creates organizations automatically, simpler flow, good for admin/owner accounts
- **Project-based** (`/api/auth/local/*`): Requires existing project, project-specific authentication, includes rate limiting and captcha verification, better for end-user authentication

### 3. OAuth Authentication

#### Supported Providers
- Google, GitHub, Facebook, Microsoft, Apple, LinkedIn, Discord, Twitter, etc.

#### OAuth Flow Example (Google)

```javascript
// Step 1: Initiate OAuth
// Redirect user to:
GET /api/auth/oauth/google?projectId=507f1f77bcf86cd799439011

// User is redirected to Google consent screen
// After consent, Google redirects to:
GET /api/auth/oauth/google/callback?code={authorization_code}

// Step 2: Backend processes OAuth callback
// Backend automatically:
// 1. Exchanges code for access token
// 2. Fetches user profile from Google
// 3. Creates/updates user in database
// 4. Generates JWT token
// 5. Redirects to frontend with token

// Frontend receives redirect:
https://yourapp.com/auth/callback?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// Step 3: Use token for authenticated requests
Authorization: Bearer {token}

// Response (from OAuth callback redirect)
// Frontend receives:
https://yourapp.com/auth/callback?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...&user={"id":"507f1f77bcf86cd799439012","email":"john.doe@example.com"}
```

#### OAuth Provider Configuration

```javascript
// Configure OAuth provider in project settings
PATCH /api/projects/{projectId}
Authorization: Bearer {admin_token}
Content-Type: application/json

{
  "authProviders": {
    "google": {
      "enabled": true,
      "clientId": "your-google-client-id",
      "clientSecret": "your-google-client-secret",
      "callbackUrl": "https://api.mudbase.com/api/auth/oauth/google/callback"
    },
    "github": {
      "enabled": true,
      "clientId": "your-github-client-id",
      "clientSecret": "your-github-client-secret"
    }
  }
}
```

### 4. Magic Link Authentication

```javascript
// Step 1: Request Magic Link
POST /api/auth/magic-link/send
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "projectId": "507f1f77bcf86cd799439011",
  "redirectUrl": "https://yourapp.com/auth/verify"
}

// Response
{
  "success": true,
  "message": "Magic link sent to email"
}

// Step 2: User clicks link in email
// Email contains: https://api.mudbase.com/api/auth/magic-link/verify?token={magic_token}

// Step 3: Verify Magic Link
GET /api/auth/magic-link/verify?token={magic_token}&projectId={projectId}

// Response (redirects to frontend with token)
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

### 5. OTP Authentication (SMS/Email)

```javascript
// Step 1: Request OTP
POST /api/auth/otp/send
Content-Type: application/json

{
  "identifier": "john.doe@example.com", // or phone number
  "method": "email", // or "sms" or "auto"
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "message": "OTP sent to email",
  "expiresIn": 300 // 5 minutes
}

// Step 2: Verify OTP
POST /api/auth/otp/verify
Content-Type: application/json

{
  "identifier": "john.doe@example.com",
  "otp": "123456",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

### 6. Two-Factor Authentication (2FA)

```javascript
// Step 1: Enable 2FA
POST /api/auth/2fa/setup
Authorization: Bearer {token}
Content-Type: application/json

// Response
{
  "secret": "JBSWY3DPEHPK3PXP",
  "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..."
}

// Step 2: Verify and Enable
POST /api/auth/2fa/verify
Authorization: Bearer {token}
Content-Type: application/json

{
  "token": "123456" // TOTP code from authenticator app
}

// Step 3: Login with 2FA
POST /api/auth/local/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response (requires 2FA)
{
  "requires2FA": true,
  "tempToken": "temp-token-for-2fa-verification"
}

// Step 4: Verify 2FA
POST /api/auth/2fa/verify-login
Content-Type: application/json

{
  "tempToken": "temp-token-for-2fa-verification",
  "token": "123456"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

---

## Organization & Project Setup

### Complete Setup Workflow

```javascript
// Step 1: Create Organization
POST /api/orgs
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Acme Corporation",
  "slug": "acme-corp" // auto-generated if not provided
}

// Response
{
  "success": true,
  "org": {
    "_id": "507f1f77bcf86cd799439013",
    "name": "Acme Corporation",
    "slug": "acme-corp",
    "members": ["507f1f77bcf86cd799439012"],
    "createdAt": "2024-01-15T10:00:00Z"
  }
}

// Step 2: Create Project
POST /api/projects
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "My Awesome App",
  "orgId": "507f1f77bcf86cd799439013",
  "description": "A revolutionary app"
}

// Response
{
  "success": true,
  "project": {
    "_id": "507f1f77bcf86cd799439011",
    "name": "My Awesome App",
    "slug": "my-awesome-app",
    "org": "507f1f77bcf86cd799439013",
    "apiKey": "pk_live_abc123...",
    // NOTE: secretKey is NEVER returned in responses
    "settings": {
      "auth": {
        "requireEmailVerification": true,
        "sessionTimeout": 1800000
      }
    }
  }
}

// Step 3: Configure Project Settings
PATCH /api/projects/507f1f77bcf86cd799439011
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "settings": {
    "auth": {
      "providers": {
        "local": { "enabled": true },
        "google": { "enabled": true },
        "github": { "enabled": true },
        "magicLink": { "enabled": true },
        "otp": { "enabled": true }
      },
      "requireEmailVerification": true,
      "sessionTimeout": 1800000,
      "passwordPolicy": {
        "minLength": 8,
        "requireUppercase": true,
        "requireNumbers": true,
        "requireSpecialChars": true
      }
    },
    "database": {
      "collections": {
        "users": { "enabled": true },
        "products": { "enabled": true },
        "orders": { "enabled": true }
      }
    },
    "storage": {
      "maxFileSize": 10485760, // 10MB
      "allowedTypes": ["image/jpeg", "image/png", "application/pdf"]
    }
  }
}

// Step 4: Set up Payment Gateway (for billing)
POST /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "provider": "paystack",
  "publicKey": "pk_test_abc123...",
  "secretKey": "[SECRET_KEY_NOT_RETURNED]", // Only sent during creation, never returned
  "config": {
    "currency": "NGN",
    "webhookUrl": "https://api.mudbase.com/api/billing/webhooks/paystack"
  }
}

// Response
{
  "message": "Payment gateway created successfully",
  "gateway": {
    "_id": "507f1f77bcf86cd799439014",
    "provider": "paystack",
    "status": "pending"
    // NOTE: secretKey and webhookSecret are NEVER returned in responses
  }
}

// Step 5: Activate Payment Gateway
PATCH /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "status": "active"
}
```

---

## Wallet as a Service Workflows

### Important Security Notes

1. **Private Key Generation**: Each wallet gets its own unique private key. Keys are generated per wallet, not shared per project or organization.
2. **Private Key Visibility**: Private keys are ONLY shown once in the `/api/wallet/generate-key` endpoint response. They are NEVER returned in:
   - Wallet creation responses
   - Wallet listing responses
   - Wallet balance responses
   - Any other wallet-related endpoints
3. **Encryption**: All private keys are encrypted using AES-256-GCM before storage in the database.
4. **Key Storage**: Users must save the private key securely when generated. It cannot be retrieved later.

### Understanding Wallet Encryption

**Two Types of Keys**:

1. **Project Encryption Key** (`walletEncryptionKey`):
   - One per project (auto-generated)
   - Used to encrypt/decrypt wallet private keys for database storage
   - Provides additional security layer and project isolation
   - **NOT used for signing transactions**

2. **Wallet Private Keys**:
   - One unique key per wallet (generated per wallet)
   - The actual cryptocurrency private key
   - Used to sign blockchain transactions
   - Encrypted using project encryption key before storage

**Transaction Flow**:
- When making a transaction, the wallet's private key is decrypted using the project encryption key
- The decrypted wallet private key is then used to sign the transaction
- The project encryption key is NEVER used to sign transactions - only the wallet's private key

For detailed explanation, see `docs/WALLET_ENCRYPTION_EXPLAINED.md`.

### Supported Cryptocurrencies
- **BTC** (Bitcoin) - Bech32 addresses (bc1...)
- **ETH** (Ethereum)
- **BNB** (Binance Smart Chain)
- **SOL** (Solana)
- **TRX** (Tron)
- **LTC** (Litecoin)
- **USDT** (Tether - Multi-network: ETH, TRX, BSC, SOL, POLYGON)

### Platform Fee Structure (Hybrid Model)

**Formula**: `Platform Fee = max(1% * amount, minimumFee)`

| Currency | Minimum Fee | Minimum Withdrawal | Notes |
|----------|-------------|-------------------|-------|
| BTC | 0.00012 BTC (~$8) | 0.0002 BTC (~$13) | Updated based on market rates |
| ETH | 0.001 ETH (~$2.50) | 0.001 ETH (~$2.50) | Increased to protect against gas spikes |
| BNB | 0.0002 BNB (~$0.13) | 0.001 BNB (~$0.65) | Updated based on market rates |
| SOL | 0.008 SOL (~$1.20) | 0.01 SOL (~$1.50) | Platform as feePayer (no pre-fund) |
| TRX | 1 TRX (~$0.05) | 5 TRX (~$0.25) | Lowered for micro-transactions support |
| LTC | 0.0001 LTC (~$0.01) | 0.001 LTC (~$0.10) | Updated based on market rates |
| USDT-ETH | $4 fixed | 5 USDT | Covers gas spikes |
| USDT-BSC | $0.75 fixed | 2 USDT | Scaled with network cost + profit margin |
| USDT-TRX | $0.50 fixed | 2 USDT | Scaled with network cost + profit margin |
| USDT-SOL | $0.15 fixed | 2 USDT | Scaled with network cost + profit margin |

**Profitability Guardrails**: Transactions are rejected if platform fee < estimated network cost + pre-fund cost.

**Important Notes**:
- **ETH Gas Spikes**: ETH minimum fee (0.001 ETH) protects against gas spikes, but network conditions may still cause fees to exceed minimum during extreme congestion. Consider implementing dynamic fee adjustments for ETH during high gas periods.
- **USDT Fees**: Fees on non-ETH chains (BSC, TRX, SOL) are scaled with network cost + profit margin rather than flat rates for better user experience and competitiveness.
- **Micro-Transactions**: TRX minimum withdrawal lowered to 5 TRX to support small users and micro-transactions.

For detailed fee structure and rationale, see [FEE_STRUCTURE_UPDATED.md](./FEE_STRUCTURE_UPDATED.md)

### 1. Generate Private Key (Dashboard Endpoint)

**Important**: Each wallet gets its own unique private key. Keys are generated per wallet, not per project. The private key is only shown once in this response and must be saved securely.

```javascript
// Generate a new key pair for any supported currency
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "network": null // Only required for USDT
}

// Response
{
  "success": true,
  "data": {
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "privateKey": "L5EZftvrYaSudiozVRzTqLcHLNDoVn7H5HSfM9BAN6pMA5zoi3WZ",
    "publicKey": "02a1633cafcc01ebfb6d78e39f687a1f0995c62fc95f51ead10a02ee0be551b5fb"
  },
  "warning": "This private key is shown only once. Store it securely."
}

// For USDT with network
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "USDT",
  "network": "ETH" // Options: ETH, TRX, BSC, SOL, POLYGON
}

// Response
{
  "success": true,
  "data": {
    "currency": "USDT",
    "network": "ETH",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "privateKey": "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
    "publicKey": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
  },
  "warning": "This private key is shown only once. Store it securely."
}
```

### 2. Create Wallet with Generated Private Key

**Security Note**: Private keys are NEVER returned in wallet creation responses. They are encrypted and stored securely. Only the address and wallet metadata are returned.

```javascript
// User can set the generated private key from dashboard
POST /api/wallet/create
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "privateKey": "[PRIVATE_KEY_FROM_GENERATE_KEY_ENDPOINT]",
  "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh"
}

// Response
{
  "success": true,
  "message": "BTC wallet created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439015",
    "user": "507f1f77bcf86cd799439012",
    "org": "507f1f77bcf86cd799439013",
    "project": "507f1f77bcf86cd799439011",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "balance": "0",
    "isCustomKey": true,
    "isActive": true,
    "createdAt": "2024-01-15T10:00:00Z",
    "updatedAt": "2024-01-15T10:00:00Z"
  }
}
```

### 3. Create Wallet with Auto-Generated Key

```javascript
// Let system generate the key pair
POST /api/wallet/create
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "ETH"
}

// Response
{
  "success": true,
  "message": "ETH wallet created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439016",
    "user": "507f1f77bcf86cd799439012",
    "org": "507f1f77bcf86cd799439013",
    "project": "507f1f77bcf86cd799439011",
    "currency": "ETH",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "balance": "0",
    "isCustomKey": false,
    "isActive": true,
    "createdAt": "2024-01-15T10:00:00Z",
    "updatedAt": "2024-01-15T10:00:00Z"
  }
}
```

### 4. Get User Wallets

```javascript
// Get all wallets for authenticated user
GET /api/wallet
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": [
    {
      "_id": "507f1f77bcf86cd799439015",
      "currency": "BTC",
      "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
      "balance": "0.05",
      "isCustomKey": true,
      "isActive": true,
      "project": "507f1f77bcf86cd799439011",
      "createdAt": "2024-01-15T10:00:00Z"
    },
    {
      "_id": "507f1f77bcf86cd799439016",
      "currency": "ETH",
      "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
      "balance": "2.5",
      "isCustomKey": false,
      "isActive": true,
      "project": "507f1f77bcf86cd799439011",
      "createdAt": "2024-01-15T10:00:00Z"
    }
  ]
}
```

### 4.5. Get Wallet Private Key

**Security Warning**: This endpoint returns the decrypted private key. Use only when you need to export your wallet. The private key is sensitive and should be kept secure.

```javascript
// Get private key for a specific wallet
GET /api/wallet/{walletId}/private-key
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "walletId": "507f1f77bcf86cd799439015",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "privateKey": "L5EZftvrYaSudiozVRzTqLcHLNDoVn7H5HSfM9BAN6pMA5zoi3WZ",
    "isCustomKey": true
  },
  "warning": "Keep this private key secure and never share it. Anyone with access to this key can control your wallet.",
  "security": {
    "accessedAt": "2024-01-15T11:20:00Z",
    "accessedBy": "507f1f77bcf86cd799439012"
  }
}
```

**Access Control**:
- User can only retrieve private keys for wallets they own
- Wallet must belong to the user's organization
- All access is logged for security audit

### 6. Get Wallet Balance

```javascript
// Get balance for specific wallet
GET /api/wallet/507f1f77bcf86cd799439015/balance
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "walletId": "507f1f77bcf86cd799439015",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "balance": "0.05",
    "balanceInUSD": 2500.00,
    "lastSyncedAt": "2024-01-15T10:30:00Z"
  }
}
```

### 7. Withdraw Funds (Async Processing)

**Important**: Withdrawals are processed asynchronously. The API returns immediately with a `queued` status. You must check the transaction status to see when it's confirmed. Real blockchain confirmations take time (BTC: 10-60 minutes, ETH: 15-45 seconds, SOL: 5-10 seconds, etc.).

**Fee Model**: Platform fees use a hybrid model: `max(1% * amount, minimumFee)`. Project fees (optional) are added on top if configured. Minimum fees are enforced per currency to ensure profitability.

```javascript
// Withdraw from wallet
POST /api/wallet/507f1f77bcf86cd799439015/withdraw
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "toAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "amount": 0.01,
  "feeRate": 10 // For BTC/LTC only
}

// Response (Returns Immediately - Async Processing)
{
  "success": true,
  "message": "Withdrawal is processing",
  "data": {
    "transactionId": "507f1f77bcf86cd799439027",
    "status": "queued",
    "amount": 0.01,
    "platformFee": 0.00012, // max(1% * 0.01, 0.00012 BTC minimum)
    "projectFee": 0.00005, // Optional: if project fee is configured
    "totalFee": 0.00017, // Platform fee + project fee
    "message": "Withdrawal is processing. Check transaction status for updates."
  }
}

// Check Transaction Status
GET /api/wallet/transactions/507f1f77bcf86cd799439027
Authorization: Bearer {user_token}

// Response (During Processing)
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439027",
    "status": "prefunding", // or "processing", "broadcasted"
    "mainTxStatus": "pending",
    "mainTxHash": null,
    "platformFee": 0.0005,
    "createdAt": "2024-01-15T10:35:00Z"
  }
}

// Response (After Confirmation)
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439027",
    "status": "confirmed",
    "mainTxHash": "abc123def456ghi789jkl012mno345pqr678stu901vwx234yz",
    "mainTxStatus": "confirmed",
    "refundTxHash": "xyz789abc123def456ghi789jkl012mno345pqr678",
    "refundStatus": "confirmed",
    "networkFee": 0.0001,
    "platformFee": 0.0005,
    "completedAt": "2024-01-15T10:45:00Z"
  }
}
```

**Transaction Status Flow**:
- `queued` → Transaction added to processing queue
- `prefunding` → Platform sending pre-fund (if needed)
- `processing` → User transaction being prepared
- `broadcasted` → Transaction broadcast to blockchain
- `confirmed` → Transaction confirmed on blockchain
- `completed` → All operations completed successfully
- `failed` → Transaction failed (check error field)
- `partial` → User tx succeeded but refund failed (CRITICAL - requires manual intervention)

### 8. Validate Address

```javascript
// Validate cryptocurrency address
POST /api/wallet/validate-address
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "currency": "BTC"
}

// Response
{
  "success": true,
  "data": {
    "isValid": true,
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "format": "bech32"
  }
}
```

### 9. Get Supported Currencies

```javascript
// Get list of supported currencies
GET /api/wallet/currencies
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": [
    {
      "code": "BTC",
      "name": "Bitcoin",
      "network": "mainnet"
    },
    {
      "code": "ETH",
      "name": "Ethereum",
      "network": "mainnet"
    },
    {
      "code": "BNB",
      "name": "Binance Coin",
      "network": "bsc"
    },
    {
      "code": "LTC",
      "name": "Litecoin",
      "network": "mainnet"
    },
    {
      "code": "SOL",
      "name": "Solana",
      "network": "mainnet"
    },
    {
      "code": "TRX",
      "name": "Tron",
      "network": "mainnet"
    },
    {
      "code": "USDT",
      "name": "Tether",
      "network": "ethereum"
    }
  ]
}
```

---

## Billing & Payment Gateway Workflows

### 1. Setup Payment Gateway

```javascript
// Create payment gateway account
POST /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "provider": "paystack", // or "flutterwave", "monnify", "interswitch"
  "publicKey": "pk_test_abc123...",
  "secretKey": "[SECRET_KEY_NOT_RETURNED]",
  "config": {
    "currency": "NGN",
    "webhookUrl": "https://api.mudbase.com/api/billing/webhooks/paystack"
  }
}

// Response
{
  "message": "Payment gateway created successfully",
  "gateway": {
    "_id": "507f1f77bcf86cd799439017",
    "provider": "paystack",
    "status": "pending",
    "publicKey": "pk_test_abc123...",
    "createdAt": "2024-01-15T10:40:00Z"
  }
}

// Activate gateway
PATCH /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "status": "active"
}
```

### 2. Create Subscription Plan

```javascript
// Create a plan for your project
POST /api/billing/projects/507f1f77bcf86cd799439011/plans
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Pro Plan",
  "description": "Professional plan with advanced features",
  "pricing": {
    "type": "recurring",
    "monthly": {
      "amount": 5000, // in kobo (50 NGN)
      "currency": "NGN"
    },
    "yearly": {
      "amount": 50000, // in kobo (500 NGN)
      "currency": "NGN"
    },
    "usage": [
      {
        "metric": "api_calls",
        "unitPrice": 0.1, // per API call
        "includedUnits": 10000,
        "currency": "NGN"
      }
    ]
  },
  "features": [
    {
      "name": "api_access",
      "description": "Full API access",
      "included": true
    },
    {
      "name": "storage",
      "description": "100GB storage",
      "included": true,
      "limit": 107374182400 // 100GB in bytes
    }
  ],
  "limits": {
    "apiCalls": 100000,
    "storage": 107374182400,
    "bandwidth": 1073741824000
  },
  "trial": {
    "enabled": true,
    "days": 7
  },
  "isActive": true,
  "isDefault": false
}

// Response
{
  "message": "Plan created successfully",
  "plan": {
    "_id": "507f1f77bcf86cd799439018",
    "name": "Pro Plan",
    "description": "Professional plan with advanced features",
    "pricing": {
      "type": "recurring",
      "monthly": {
        "amount": 5000,
        "currency": "NGN"
      },
      "yearly": {
        "amount": 50000,
        "currency": "NGN"
      }
    },
    "features": [
      {
        "name": "api_access",
        "description": "Full API access",
        "included": true
      },
      {
        "name": "storage",
        "description": "100GB storage",
        "included": true,
        "limit": 107374182400
      }
    ],
    "isActive": true,
    "createdAt": "2024-01-15T10:40:00Z"
  }
}
```

### 3. Customer Checkout Flow

```javascript
// Step 1: Get available plans (Public API)
GET /api/billing/public/projects/507f1f77bcf86cd799439011/plans

// Response
{
  "plans": [
    {
      "_id": "507f1f77bcf86cd799439018",
      "name": "Pro Plan",
      "description": "Professional plan with advanced features",
      "pricing": {
        "monthly": {
          "amount": 5000,
          "currency": "NGN"
        },
        "yearly": {
          "amount": 50000,
          "currency": "NGN"
        }
      },
      "features": [ ... ]
    }
  ]
}

// Step 2: Create checkout session
POST /api/billing/public/projects/507f1f77bcf86cd799439011/checkout
Content-Type: application/json

{
  "planId": "507f1f77bcf86cd799439018",
  "billingCycle": "monthly",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  },
  "successUrl": "https://yourapp.com/billing/success",
  "cancelUrl": "https://yourapp.com/billing/cancel"
}

// Response
{
  "success": true,
  "data": {
    "checkoutUrl": "https://paystack.com/pay/abc123...",
    "reference": "mudbase_1705315200_abc123def456",
    "amount": 5000,
    "currency": "NGN"
  }
}

// Step 3: Redirect customer to authorizationUrl
// Customer completes payment on Paystack/Flutterwave

// Step 4: Payment Gateway redirects to callback URL
// Backend automatically verifies payment and creates subscription

// Step 5: Verify payment (if needed manually)
POST /api/billing/public/projects/507f1f77bcf86cd799439011/verify-payment
Content-Type: application/json

{
  "reference": "mudbase_1705315200_abc123def456",
  "orgId": "507f1f77bcf86cd799439013"
}

// Response
{
  "success": true,
  "message": "Payment verified and subscription created",
  "data": {
    "subscription": {
      "_id": "507f1f77bcf86cd799439019",
      "status": "active",
      "plan": {
        "_id": "507f1f77bcf86cd799439018",
        "name": "Pro Plan"
      },
      "customerEmail": "customer@example.com",
      "currentPeriodEnd": "2024-02-15T10:45:00Z",
      "billingCycle": "monthly"
    }
  }
}
```

### 4. Check Subscription Status

```javascript
// Check customer subscription (Public API)
GET /api/billing/public/projects/507f1f77bcf86cd799439011/subscription?email=customer@example.com

// Response
{
  "hasSubscription": true,
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active",
    "plan": {
      "_id": "507f1f77bcf86cd799439018",
      "name": "Pro Plan",
      "price": 5000,
      "currency": "NGN"
    },
    "customerEmail": "customer@example.com",
    "currentPeriodEnd": "2024-02-15T10:45:00Z",
    "billingCycle": "monthly",
    "createdAt": "2024-01-15T10:45:00Z"
  }
}
```

### 5. Check Feature Access

```javascript
// Check if customer has access to a feature
GET /api/billing/public/projects/507f1f77bcf86cd799439011/feature-access?email=customer@example.com&feature=api_access

// Response
{
  "hasAccess": true,
  "reason": "Active subscription",
  "plan": {
    "_id": "507f1f77bcf86cd799439018",
    "name": "Pro Plan"
  },
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active"
  }
}
```

### 6. Record Usage (Metered Billing)

```javascript
// Record usage for metered billing
POST /api/billing/public/projects/507f1f77bcf86cd799439011/usage
Content-Type: application/json

{
  "email": "customer@example.com",
  "metric": "api_calls",
  "quantity": 150
}

// Response
{
  "success": true,
  "message": "Usage recorded successfully"
}
```

### 7. Cancel Subscription

```javascript
// Cancel subscription
POST /api/billing/subscriptions/507f1f77bcf86cd799439019/cancel
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "cancelImmediately": false // Cancel at period end
}

// Response
{
  "message": "Subscription canceled successfully",
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active",
    "cancelAtPeriodEnd": true,
    "currentPeriodEnd": "2024-02-15T10:45:00Z",
    "canceledAt": "2024-01-15T11:00:00Z"
  }
}
```

### 8. Payment Gateway Webhook

```javascript
// Payment gateway sends webhook on payment events
POST /api/billing/webhooks/paystack
Content-Type: application/json
X-Paystack-Signature: {signature}

{
  "event": "charge.success",
  "data": {
    "reference": "mudbase_1705315200_abc123def456",
    "amount": 5000,
    "currency": "NGN",
    "customer": {
      "email": "customer@example.com"
    },
    "metadata": {
      "projectId": "507f1f77bcf86cd799439011",
      "planId": "507f1f77bcf86cd799439018",
      "billingCycle": "monthly",
      "orgId": "507f1f77bcf86cd799439013"
    }
  }
}

// Backend automatically:
// 1. Verifies webhook signature
// 2. Verifies payment
// 3. Creates subscription
// 4. Sends confirmation email
// 5. Triggers project webhook
```

---

## Database Operations

### 1. Create Collection Schema

```javascript
// Define collection schema
POST /api/projects/507f1f77bcf86cd799439011/schemas
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "products",
  "fields": [
    {
      "name": "name",
      "type": "string",
      "required": true,
      "indexed": true
    },
    {
      "name": "price",
      "type": "number",
      "required": true
    },
    {
      "name": "description",
      "type": "string"
    },
    {
      "name": "category",
      "type": "string",
      "indexed": true
    },
    {
      "name": "inStock",
      "type": "boolean",
      "default": true
    },
    {
      "name": "createdAt",
      "type": "date",
      "default": "now"
    }
  ],
  "indexes": [
    {
      "fields": ["category", "price"],
      "unique": false
    }
  ]
}

// Response
{
  "success": true,
  "collection": {
    "_id": "507f1f77bcf86cd799439020",
    "name": "products",
    "slug": "products",
    "project": "507f1f77bcf86cd799439011",
    "fields": [
      {
        "name": "name",
        "type": "string",
        "required": true,
        "indexed": true
      },
      {
        "name": "price",
        "type": "number",
        "required": true
      },
      {
        "name": "description",
        "type": "string"
      },
      {
        "name": "category",
        "type": "string",
        "indexed": true
      },
      {
        "name": "inStock",
        "type": "boolean",
        "default": true
      }
    ],
    "createdAt": "2024-01-15T10:50:00Z"
  }
}
```

### 2. Create Document

```javascript
// Create a document in collection
POST /api/projects/507f1f77bcf86cd799439011/data/products
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Laptop",
  "price": 999.99,
  "description": "High-performance laptop",
  "category": "electronics",
  "inStock": true
}

// Response
{
  "message": "Data created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439021",
    "name": "Laptop",
    "price": 999.99,
    "description": "High-performance laptop",
    "category": "electronics",
    "inStock": true,
    "createdAt": "2024-01-15T10:55:00Z",
    "updatedAt": "2024-01-15T10:55:00Z"
  }
}
```

### 3. Query Documents

```javascript
// Get all documents with filters
GET /api/projects/507f1f77bcf86cd799439011/data/products?category=electronics&price[gte]=500&limit=10&page=1
Authorization: Bearer {user_token}

// Response
{
  "data": [
    {
      "_id": "507f1f77bcf86cd799439021",
      "name": "Laptop",
      "price": 999.99,
      "description": "High-performance laptop",
      "category": "electronics",
      "inStock": true,
      "createdAt": "2024-01-15T10:55:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 1,
    "totalPages": 1
  }
}

// Advanced query
POST /api/projects/507f1f77bcf86cd799439011/data/products/query
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "filter": {
    "$and": [
      { "category": "electronics" },
      { "price": { "$gte": 500 } },
      { "inStock": true }
    ]
  },
  "sort": { "price": -1 },
  "limit": 10,
  "skip": 0
}
```

### 4. Update Document

```javascript
// Update document
PATCH /api/projects/507f1f77bcf86cd799439011/data/products/507f1f77bcf86cd799439021
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "price": 899.99,
  "inStock": false
}

// Response
{
  "message": "Data updated successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439021",
    "name": "Laptop",
    "price": 899.99,
    "description": "High-performance laptop",
    "category": "electronics",
    "inStock": false,
    "createdAt": "2024-01-15T10:55:00Z",
    "updatedAt": "2024-01-15T11:00:00Z"
  }
}
```

### 5. Delete Document

```javascript
// Delete document
DELETE /api/projects/507f1f77bcf86cd799439011/data/products/507f1f77bcf86cd799439021
Authorization: Bearer {user_token}

// Response
{
  "message": "Data deleted successfully"
}
```

---

## Real-time Chat System

### 1. Create Chat

```javascript
// Create a new chat
POST /api/projects/507f1f77bcf86cd799439011/chats
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Team Discussion",
  "type": "group", // or "direct"
  "participants": [
    "507f1f77bcf86cd799439012",
    "507f1f77bcf86cd799439022"
  ],
  "metadata": {
    "projectId": "507f1f77bcf86cd799439011"
  }
}

// Response
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439023",
    "name": "Team Discussion",
    "type": "group",
    "participants": [
      {
        "user": "507f1f77bcf86cd799439012",
        "role": "admin",
        "joinedAt": "2024-01-15T11:05:00Z"
      },
      {
        "user": "507f1f77bcf86cd799439022",
        "role": "member",
        "joinedAt": "2024-01-15T11:05:00Z"
      }
    ],
    "createdBy": "507f1f77bcf86cd799439012",
    "project": "507f1f77bcf86cd799439011",
    "createdAt": "2024-01-15T11:05:00Z"
  }
}
```

### 2. Send Message (HTTP)

```javascript
// Send message via HTTP
POST /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "content": "Hello team! How's everyone doing?",
  "type": "text" // or "image", "file", etc.
}

// Response
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439024",
    "content": "Hello team! How's everyone doing?",
    "type": "text",
    "sender": "507f1f77bcf86cd799439012",
    "chat": "507f1f77bcf86cd799439023",
    "project": "507f1f77bcf86cd799439011",
    "createdAt": "2024-01-15T11:06:00Z",
    "updatedAt": "2024-01-15T11:06:00Z"
  }
}
```

### 3. Real-time Messaging (Socket.IO)

```javascript
// Client-side Socket.IO connection
import io from 'socket.io-client';

const socket = io('https://api.mudbase.com', {
  auth: {
    token: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
  }
});

// Join chat room
socket.emit('chat:join', {
  chatId: '507f1f77bcf86cd799439023',
  projectId: '507f1f77bcf86cd799439011'
});

// Send message via Socket.IO
socket.emit('chat:message:send', {
  chatId: '507f1f77bcf86cd799439023',
  content: 'Hello from real-time!',
  type: 'text'
});

// Listen for new messages
socket.on('chat:message:new', (data) => {
  console.log('New message:', data);
  // {
  //   message: {
  //     _id: '507f1f77bcf86cd799439025',
  //     content: 'Hello from real-time!',
  //     sender: { ... },
  //     createdAt: '2024-01-15T11:07:00Z'
  //   },
  //   chatId: '507f1f77bcf86cd799439023'
  // }
});

// Typing indicator
socket.emit('chat:typing', {
  chatId: '507f1f77bcf86cd799439023',
  isTyping: true
});

socket.on('chat:typing', (data) => {
  console.log('User typing:', data);
  // {
  //   userId: '507f1f77bcf86cd799439012',
  //   chatId: '507f1f77bcf86cd799439023',
  //   isTyping: true
  // }
});

// Voice/Video call events
socket.emit('chat:call:initiate', {
  chatId: '507f1f77bcf86cd799439023',
  type: 'video' // or 'voice'
});

socket.on('chat:call:incoming', (data) => {
  console.log('Incoming call:', data);
});

socket.emit('chat:call:accept', {
  callId: 'call_abc123'
});

socket.emit('chat:call:reject', {
  callId: 'call_abc123'
});

socket.emit('chat:call:end', {
  callId: 'call_abc123'
});
```

### 4. Edit Message

```javascript
// Edit message
PATCH /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "content": "Hello team! How's everyone doing? (edited)"
}

// Socket.IO event also emitted: 'chat:message:updated'
```

### 5. Delete Message

```javascript
// Delete message
DELETE /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024
Authorization: Bearer {user_token}

// Socket.IO event also emitted: 'chat:message:deleted'
```

### 6. Add Reaction

```javascript
// Add reaction to message
POST /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024/reactions
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "emoji": "👍"
}

// Socket.IO event also emitted: 'chat:message:reaction'
```

---

## Integration System

### 1. Create Integration

```javascript
// Create integration
POST /api/projects/507f1f77bcf86cd799439011/integrations
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Twitter API Integration",
  "provider": "twitter",
  "category": "social",
    "settings": {
      "isActive": true
      // NOTE: API keys, secrets, and tokens are encrypted and never returned in responses
    },
  "config": {
    "rateLimit": 100,
    "timeout": 5000
  }
}

// Response
{
  "integration": {
    "_id": "507f1f77bcf86cd799439026",
    "name": "Twitter API Integration",
    "provider": "twitter",
    "category": "social",
    "project": "507f1f77bcf86cd799439011",
    "settings": {
      "isActive": true
    },
    "createdAt": "2024-01-15T11:10:00Z",
    "updatedAt": "2024-01-15T11:10:00Z"
  }
}
```

### 2. Execute Integration

```javascript
// Execute integration
POST /api/projects/507f1f77bcf86cd799439011/integrations/507f1f77bcf86cd799439026/execute
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "method": "POST",
  "endpoint": "/2/tweets",
  "body": {
    "text": "Hello from MUDBASE!"
  },
  "headers": {
    "Content-Type": "application/json"
  }
}

// Response
{
  "success": true,
  "data": {
    "status": 201,
    "data": {
      "id": "1234567890",
      "text": "Hello from MUDBASE!",
      "created_at": "2024-01-15T11:12:00Z"
    },
    "headers": {
      "content-type": "application/json",
      "x-rate-limit-remaining": "299"
    }
  },
  "usage": {
    "apiCalls": 1,
    "timestamp": "2024-01-15T11:12:00Z"
  }
}
```

### 3. Get Integration Usage Stats

```javascript
// Get usage statistics
GET /api/projects/507f1f77bcf86cd799439011/integrations/507f1f77bcf86cd799439026/usage?period=month
Authorization: Bearer {user_token}

// Response
{
  "stats": {
    "totalCalls": 1250,
    "successfulCalls": 1200,
    "failedCalls": 50,
    "successRate": 96.0,
    "period": "month",
    "breakdown": [
      {
        "date": "2024-01-15",
        "calls": 45,
        "successful": 43,
        "failed": 2
      }
    ]
  }
}
```

---

## Complete End-to-End Scenarios

### Scenario 1: E-commerce Platform Setup

```javascript
// Step 1: Register Admin User
POST /api/auth/local/register
{
  "email": "admin@ecommerce.com",
  "password": "SecurePass123!",
  "firstName": "Admin",
  "lastName": "User"
}

// Step 2: Create Organization
POST /api/orgs
Authorization: Bearer {admin_token}
{
  "name": "Ecommerce Corp"
}

// Step 3: Create Project
POST /api/projects
Authorization: Bearer {admin_token}
{
  "name": "Ecommerce Platform",
  "orgId": "{org_id}"
}

// Step 4: Setup Payment Gateway
POST /api/billing/orgs/{org_id}/payment-gateway
Authorization: Bearer {admin_token}
{
  "provider": "paystack",
  "publicKey": "pk_test_...",
  "secretKey": "sk_test_..."
}

// Step 5: Create Product Schema
POST /api/projects/{project_id}/schemas
Authorization: Bearer {admin_token}
{
  "name": "products",
  "fields": [
    { "name": "name", "type": "string", "required": true },
    { "name": "price", "type": "number", "required": true },
    { "name": "stock", "type": "number", "required": true }
  ]
}

// Step 6: Create Order Schema
POST /api/projects/{project_id}/schemas
Authorization: Bearer {admin_token}
{
  "name": "orders",
  "fields": [
    { "name": "customerId", "type": "string", "required": true },
    { "name": "products", "type": "array", "required": true },
    { "name": "total", "type": "number", "required": true },
    { "name": "status", "type": "string", "default": "pending" }
  ]
}

// Step 7: Customer Registration
POST /api/auth/local/register
{
  "email": "customer@example.com",
  "password": "CustomerPass123!",
  "firstName": "John",
  "lastName": "Customer",
  "projectId": "{project_id}"
}

// Step 8: Customer Creates Order
POST /api/projects/{project_id}/data/orders
Authorization: Bearer {customer_token}
{
  "customerId": "{customer_id}",
  "products": [
    { "productId": "{product_id}", "quantity": 2 }
  ],
  "total": 1999.98,
  "status": "pending"
}

// Step 9: Process Payment
POST /api/billing/public/projects/{project_id}/checkout
{
  "planId": "{plan_id}",
  "billingCycle": "one-time",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  },
  "amount": 1999.98
}

// Step 10: Update Order Status
PATCH /api/projects/{project_id}/data/orders/{order_id}
Authorization: Bearer {admin_token}
{
  "status": "paid"
}
```

### Scenario 2: Crypto Wallet Integration

```javascript
// Step 1: User generates Bitcoin wallet key
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
{
  "currency": "BTC"
}

// Response includes privateKey (shown once only) that user must save securely

// Step 2: User creates wallet with generated key
POST /api/wallet/create
Authorization: Bearer {user_token}
{
  "currency": "BTC",
  "privateKey": "[PRIVATE_KEY_FROM_STEP_1]",
  "address": "[ADDRESS_FROM_STEP_1]"
}

// Response does NOT include privateKey - it's encrypted and stored securely

// Step 3: Check wallet balance
GET /api/wallet/{wallet_id}/balance
Authorization: Bearer {user_token}

// Step 4: Receive payment (external)
// Bitcoin sent to wallet address

// Step 5: Withdraw funds (async processing)
POST /api/wallet/{wallet_id}/withdraw
Authorization: Bearer {user_token}
{
  "toAddress": "bc1q...",
  "amount": 0.01,
  "feeRate": 10
}

// Response (returns immediately - async processing)
{
  "success": true,
  "data": {
    "transactionId": "...",
    "status": "queued",
    "platformFee": 0.0005 // max(1% * 0.01, 0.0005 BTC minimum)
  }
}

// Step 6: Check transaction status (poll until confirmed)
GET /api/wallet/transactions/{transaction_id}
Authorization: Bearer {user_token}

// Response (after confirmation - may take 10-60 minutes for BTC)
{
  "success": true,
  "data": {
    "status": "confirmed",
    "mainTxHash": "...",
    "mainTxStatus": "confirmed"
  }
}
```

### Scenario 3: SaaS Subscription Flow

```javascript
// Step 1: Setup billing plan
POST /api/billing/projects/{project_id}/plans
Authorization: Bearer {admin_token}
{
  "name": "Premium",
  "pricing": {
    "monthly": { "amount": 10000, "currency": "NGN" },
    "yearly": { "amount": 100000, "currency": "NGN" }
  },
  "features": [
    { "name": "api_access", "included": true },
    { "name": "storage", "included": true, "limit": 107374182400 }
  ]
}

// Step 2: Customer subscribes
POST /api/billing/public/projects/{project_id}/checkout
{
  "planId": "{plan_id}",
  "billingCycle": "monthly",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  }
}

// Step 3: Check subscription status
GET /api/billing/public/projects/{project_id}/subscription?email=customer@example.com

// Step 4: Check feature access
GET /api/billing/public/projects/{project_id}/feature-access?email=customer@example.com&feature=api_access

// Step 5: Record usage
POST /api/billing/public/projects/{project_id}/usage
{
  "email": "customer@example.com",
  "metric": "api_calls",
  "quantity": 150
}
```

---

## API Reference Quick Guide

### Authentication Endpoints

```
POST   /api/auth/register                - Register new user (org-based)
POST   /api/auth/login                   - Login user (org-based)
POST   /api/auth/logout                  - Logout user (org-based)
GET    /api/auth/session                 - Get current session (org-based)
POST   /api/auth/password-reset          - Request password reset (org-based)
POST   /api/auth/password-reset/{token}  - Reset password (org-based)
POST   /api/auth/local/register          - Register new user (project-based)
POST   /api/auth/local/login             - Login user (project-based)
POST   /api/auth/local/logout            - Logout user (project-based)
GET    /api/auth/local/session           - Get current session (project-based)
POST   /api/auth/local/password-reset    - Request password reset (project-based)
POST   /api/auth/local/password-reset/{token} - Reset password (project-based)
GET    /api/auth/oauth/{provider}        - Initiate OAuth
GET    /api/auth/oauth/{provider}/callback - OAuth callback
POST   /api/auth/magic-link/send         - Send magic link
GET    /api/auth/magic-link/verify        - Verify magic link
POST   /api/auth/otp/send                - Send OTP
POST   /api/auth/otp/verify              - Verify OTP
POST   /api/auth/2fa/setup               - Setup 2FA
POST   /api/auth/2fa/verify              - Verify 2FA
```

### Wallet Endpoints

```
GET    /api/wallet/currencies            - Get supported currencies
POST   /api/wallet/generate-key          - Generate key pair
POST   /api/wallet/validate-address      - Validate address
POST   /api/wallet/create                 - Create wallet
GET    /api/wallet                        - Get user wallets
GET    /api/wallet/{id}/balance          - Get wallet balance
GET    /api/wallet/{id}/private-key      - Get wallet private key (WARNING: Sensitive)
POST   /api/wallet/{id}/withdraw          - Withdraw funds
```

### Billing Endpoints

```
POST   /api/billing/orgs/{orgId}/payment-gateway - Create gateway
GET    /api/billing/orgs/{orgId}/payment-gateway - Get gateway status
POST   /api/billing/projects/{id}/plans         - Create plan
GET    /api/billing/projects/{id}/plans         - Get plans
GET    /api/billing/public/projects/{id}/plans  - Get public plans
POST   /api/billing/public/projects/{id}/checkout - Create checkout
POST   /api/billing/public/projects/{id}/verify-payment - Verify payment
GET    /api/billing/public/projects/{id}/subscription - Check subscription
GET    /api/billing/public/projects/{id}/feature-access - Check feature access
POST   /api/billing/public/projects/{id}/usage   - Record usage
POST   /api/billing/subscriptions/{id}/cancel   - Cancel subscription
POST   /api/billing/webhooks/{provider}         - Payment webhook
```

### Database Endpoints

```
POST   /api/projects/{id}/schemas         - Create schema
GET    /api/projects/{id}/schemas         - Get schemas
POST   /api/projects/{id}/data/{collection} - Create document
GET    /api/projects/{id}/data/{collection} - Query documents
PATCH  /api/projects/{id}/data/{collection}/{id} - Update document
DELETE /api/projects/{id}/data/{collection}/{id} - Delete document
POST   /api/projects/{id}/data/{collection}/query - Advanced query
```

### Chat Endpoints

```
POST   /api/projects/{id}/chats            - Create chat
GET    /api/projects/{id}/chats           - Get user chats
POST   /api/projects/{id}/chats/{id}/messages - Send message
GET    /api/projects/{id}/chats/{id}/messages - Get messages
PATCH  /api/projects/{id}/chats/{id}/messages/{id} - Edit message
DELETE /api/projects/{id}/chats/{id}/messages/{id} - Delete message
POST   /api/projects/{id}/chats/{id}/messages/{id}/reactions - Add reaction
```

### Integration Endpoints

```
GET    /api/projects/{id}/integrations/templates - Get templates
POST   /api/projects/{id}/integrations          - Create integration
GET    /api/projects/{id}/integrations          - Get integrations
POST   /api/projects/{id}/integrations/{id}/test - Test integration
POST   /api/projects/{id}/integrations/{id}/execute - Execute integration
GET    /api/projects/{id}/integrations/{id}/usage - Get usage stats
```

---

## Security Features

### 1. Session Management
- **Idle Timeout**: 30 minutes of inactivity
- **Rolling Sessions**: Session extends on activity
- **Secure Cookies**: HttpOnly, Secure in production

### 2. Access Control
- **RBAC**: Role-Based Access Control (owner, admin, developer, viewer)
- **Project-Level Access**: Users can only access their project's resources
- **Organization-Level Access**: Users can only access their org's resources

### 3. Encryption
- **Private Keys**: AES-256-GCM encryption at rest
- **API Credentials**: Encrypted in database
- **Password Hashing**: bcrypt with salt rounds 12

### 4. Rate Limiting
- **Global**: 100 requests per 15 minutes per IP
- **Authentication**: 5 login attempts per 15 minutes
- **API Keys**: Configurable per project

### 5. Audit Logging
- All authentication events logged
- All authorization failures logged
- All sensitive operations logged

---

## Best Practices

### 1. Authentication
- Always use HTTPS in production
- Store JWT tokens securely (httpOnly cookies recommended)
- Implement token refresh mechanism
- Use 2FA for sensitive accounts

### 2. Wallet Management
- Never log private keys
- Always validate addresses before transactions
- Use custom keys only when necessary
- Keep backup of private keys securely
- **Async Processing**: Withdrawals are processed asynchronously - check transaction status for updates
- **Real Confirmations**: System uses real blockchain confirmations (not fake delays)
- **Minimum Fees**: Enforced per currency to ensure profitability
- **Status Tracking**: Monitor transaction status through: `queued` → `prefunding` → `processing` → `broadcasted` → `confirmed` → `completed`

### 3. Billing
- Verify webhook signatures
- Handle payment failures gracefully
- Implement retry logic for failed payments
- Monitor subscription status

### 4. Database
- Use indexes for frequently queried fields
- Implement pagination for large datasets
- Validate input data
- Use transactions for critical operations

### 5. Real-time
- Handle connection failures
- Implement reconnection logic
- Use rooms for efficient message delivery
- Clean up on disconnect

---

## Error Handling

### Common Error Codes

```javascript
// Authentication Errors
401 - Unauthorized (Invalid token)
403 - Forbidden (Insufficient permissions)
429 - Too Many Requests (Rate limit exceeded)

// Validation Errors
400 - Bad Request (Invalid input)
422 - Unprocessable Entity (Validation failed)

// Resource Errors
404 - Not Found (Resource doesn't exist)
409 - Conflict (Resource already exists)

// Server Errors
500 - Internal Server Error
503 - Service Unavailable
```

### Error Response Format

```javascript
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "email": "Email is required",
      "password": "Password must be at least 8 characters"
    }
  }
}
```

---

## Project Fee Management

### Overview

Project owners can configure their own fees per currency, which are collected in addition to platform fees. These fees are accumulated and paid out via automated bi-weekly payouts.

### 1. Configure Project Fee Settings

```javascript
// Step 1: Create or update fee settings
POST /api/projects/507f1f77bcf86cd799439011/fee-settings
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "enabled": true,
  "feeAmount": 0.00005, // Must be < platform minimum fee (0.00012 BTC)
  "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "payoutThreshold": 0.001 // Minimum amount before payout
}

// Response
{
  "success": true,
  "message": "Fee settings updated successfully",
  "data": {
    "feeSettings": {
      "BTC": {
        "enabled": true,
        "feeAmount": 0.00005,
        "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "payoutThreshold": 0.001,
        "addressVerified": false
      }
    }
  }
}
```

### 2. Verify Payout Address

```javascript
// Step 1: Initiate address verification
POST /api/projects/507f1f77bcf86cd799439011/fee-settings/BTC/verify-address
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "message": "Test transaction sent. Please confirm receipt or provide transaction hash from the address.",
  "data": {
    "verificationStatus": "pending",
    "testTxHash": "abc123def456...",
    "testAmount": 0.00001,
    "instructions": "Either confirm you received the test transaction, or send a transaction FROM the payout address to prove ownership."
  }
}

// Step 2: Confirm verification
POST /api/projects/507f1f77bcf86cd799439011/fee-settings/BTC/confirm-verification
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "txHash": "xyz789abc123..." // Transaction hash proving ownership
}

// Response
{
  "success": true,
  "message": "Address verified successfully",
  "data": {
    "verified": true,
    "verifiedAt": "2024-01-15T12:00:00Z"
  }
}
```

### 3. Check Fee Balance

```javascript
// Get balance for specific currency
GET /api/projects/507f1f77bcf86cd799439011/fee-balances/BTC
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "currency": "BTC",
    "collectedAmount": 0.0005,
    "threshold": 0.001,
    "status": "accumulating",
    "nextScheduledPayoutDate": null,
    "lastPayoutDate": null,
    "totalPaidOut": 0,
    "totalCollected": 0.0005
  }
}

// Get all balances
GET /api/projects/507f1f77bcf86cd799439011/fee-balances
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "balances": [
      {
        "currency": "BTC",
        "collectedAmount": 0.0005,
        "threshold": 0.001,
        "status": "accumulating"
      },
      {
        "currency": "ETH",
        "collectedAmount": 0.002,
        "threshold": 0.001,
        "status": "ready",
        "nextScheduledPayoutDate": "2024-01-17T02:00:00Z"
      }
    ]
  }
}
```

### 4. View Payout History

```javascript
// Get payout history
GET /api/projects/507f1f77bcf86cd799439011/payout-history?limit=10&offset=0
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "payouts": [
      {
        "_id": "507f1f77bcf86cd799439030",
        "currency": "BTC",
        "grossAmount": 0.001,
        "networkFee": 0.0001,
        "netAmount": 0.0009,
        "toAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "txHash": "abc123def456...",
        "status": "completed",
        "scheduledAt": "2024-01-15T02:00:00Z",
        "processedAt": "2024-01-15T02:05:00Z",
        "confirmedAt": "2024-01-15T02:45:00Z",
        "createdAt": "2024-01-15T02:00:00Z"
      }
    ],
    "total": 1,
    "limit": 10,
    "offset": 0
  }
}
```

### 5. Request Manual Payout

```javascript
// Request manual payout (restricted: once per 30 days per currency)
POST /api/projects/507f1f77bcf86cd799439011/payouts/request-manual
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC"
}

// Response
{
  "success": true,
  "message": "Manual payout requested and processed",
  "data": {
    "payout": {
      "_id": "507f1f77bcf86cd799439031",
      "currency": "BTC",
      "netAmount": 0.0009,
      "txHash": "abc123def456...",
      "status": "completed"
    }
  }
}
```

### 6. Fee Dashboard

```javascript
// Get comprehensive fee dashboard
GET /api/projects/507f1f77bcf86cd799439011/fee-dashboard
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "feeSettings": {
      "BTC": {
        "enabled": true,
        "feeAmount": 0.00005,
        "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "addressVerified": true
      }
    },
    "balances": [
      {
        "currency": "BTC",
        "collectedAmount": 0.0005,
        "threshold": 0.001,
        "status": "accumulating",
        "totalPaidOut": 0.0009,
        "totalCollected": 0.0014
      }
    ],
    "recentPayouts": [
      {
        "_id": "507f1f77bcf86cd799439030",
        "currency": "BTC",
        "netAmount": 0.0009,
        "status": "completed",
        "createdAt": "2024-01-15T02:00:00Z"
      }
    ],
    "totalEarned": 0.0009
  }
}
```

### Important Notes

1. **Fee Validation**: Project fee must be less than platform minimum fee (cannot undercut platform)
2. **Address Verification**: Required before fees are collected (prevents fraud)
3. **Automated Payouts**: Bi-weekly schedule (Wed/Sat 2 AM UTC) when threshold is met
4. **Network Fees**: Deducted from payout amount (platform pays network fees)
5. **Manual Payouts**: Restricted to once per 30 days per currency, requires 25% of threshold
6. **Fee Collection**: Automatically collected during user withdrawals if enabled and verified

---

## Conclusion

This document provides a comprehensive overview of the MUDBASE Backend-as-a-Service platform, including:

- **Multiple Authentication Methods**: Local, OAuth, Magic Link, OTP, 2FA
- **Wallet as a Service**: Support for 7 cryptocurrencies with custom key support
- **Project Fee System**: Project owners can set their own fees with automated bi-weekly payouts
- **Billing System**: Nigerian payment gateways (Paystack, Flutterwave) with subscription management
- **Real-time Features**: Socket.IO-based chat system
- **Database Operations**: Flexible schema-based collections
- **Integration System**: 50+ API integrations
- **Security**: RBAC, encryption, rate limiting, audit logging

For more detailed API documentation, refer to the OpenAPI specification at `/api-docs`.


## Table of Contents
1. [System Architecture Overview](#system-architecture-overview)
2. [Authentication Flows](#authentication-flows)
3. [Organization & Project Setup](#organization--project-setup)
4. [Wallet as a Service Workflows](#wallet-as-a-service-workflows)
5. [Billing & Payment Gateway Workflows](#billing--payment-gateway-workflows)
6. [Database Operations](#database-operations)
7. [Real-time Chat System](#real-time-chat-system)
8. [Integration System](#integration-system)
9. [Complete End-to-End Scenarios](#complete-end-to-end-scenarios)
10. [API Reference Quick Guide](#api-reference-quick-guide)

---

## System Architecture Overview

### Core Components

```
┌────────────────────────────────────────────────────────────┐
│                    MUDBASE Backend Platform                │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Auth Layer  │  │  API Layer   │  │  Socket.IO   │      │
│  │              │  │              │  │  Real-time   │      │
│  │ • Local      │  │ • REST API   │  │ • Chat       │      │
│  │ • OAuth      │  │ • Webhooks   │  │ • Events     │      │
│  │ • Magic Link │  │ • GraphQL    │  │ • Database   │      │
│  │ • OTP        │  │              │  │              │      │
│  │ • 2FA        │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Database   │  │   Storage    │  │  Integrations│      │
│  │              │  │              │  │              │      │
│  │ • MongoDB    │  │ • File Store │  │ • 50+ APIs   │      │
│  │ • Collections│  │ • Buckets    │  │ • Webhooks   │      │
│  │ • Indexes    │  │ • CDN        │  │ • Custom     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Wallet     │  │   Billing    │  │  Security    │      │
│  │   Service    │  │              │  │              │      │
│  │              │  │ • Paystack   │  │ • RBAC       │      │
│  │ • BTC        │  │ • Flutterwave│  │ • Encryption │      │
│  │ • ETH        │  │ • Plans      │  │ • Rate Limit│       │
│  │ • SOL        │  │ • Usage      │  │ • Audit Log  │      │
│  │ • TRX        │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└────────────────────────────────────────────────────────────┘
```

### Data Flow

```
User Request → Authentication → Authorization (RBAC) → Business Logic → Database/External API → Response
                ↓
            JWT Token
                ↓
        Session Management (30min idle timeout)
                ↓
        Real-time Updates via Socket.IO
```

---

## Authentication Flows

### 1. Basic Authentication (Organization-based)

These endpoints are for organization-level authentication and create organizations automatically.

#### Registration Flow

```javascript
// Step 1: User Registration (creates organization automatically)
POST /api/auth/register
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "firstName": "John",
  "lastName": "Doe",
  "orgName": "Acme Corp" // Optional
}

// Response
{
  "message": "User registered successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner",
    "emailVerified": false,
    "org": "507f1f77bcf86cd799439013"
  }
}

// Step 2: Login
POST /api/auth/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]"
}

// Response
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  }
}

// Step 3: Get Session
GET /api/auth/session
Authorization: Bearer {token}

// Response
{
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "owner"
  },
  "authenticated": true
}

// Step 4: Password Reset Request
POST /api/auth/password-reset
Content-Type: application/json

{
  "email": "john.doe@example.com"
}

// Response
{
  "message": "Password reset email sent"
}

// Step 5: Reset Password
POST /api/auth/password-reset/{token}
Content-Type: application/json

{
  "password": "[NEW_PASSWORD]"
}

// Response
{
  "message": "Password reset successful"
}
```

### 2. Local Authentication (Project-based)

These endpoints are for project-level authentication and require a projectId.

#### Registration Flow

```javascript
// Step 1: User Registration
POST /api/auth/local/register
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "firstName": "John",
  "lastName": "Doe",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "message": "User registered successfully",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "emailVerified": false,
    "role": "developer",
    "org": "507f1f77bcf86cd799439013"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

// Step 2: Email Verification (Optional)
POST /api/auth/verify-email
Authorization: Bearer {token}
Content-Type: application/json

{
  "token": "email-verification-token-from-email"
}

// Response
{
  "success": true,
  "message": "Email verified successfully"
}

// Step 3: Login
POST /api/auth/local/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  },
  "expiresIn": "24h"
}
```

#### Password Reset (Project-based)

```javascript
// Step 1: Request password reset
POST /api/auth/local/password-reset
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "projectId": "507f1f77bcf86cd799439011" // Optional
}

// Response
{
  "message": "Password reset email sent"
}

// Step 2: Reset password
POST /api/auth/local/password-reset/{token}
Content-Type: application/json

{
  "password": "[NEW_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011" // Optional
}

// Response
{
  "message": "Password reset successful"
}
```

#### Session Management

```javascript
// Session expires after 30 minutes of inactivity
// Server automatically extends session on activity (rolling: true)

// Check current session (organization-based)
GET /api/auth/session
Authorization: Bearer {token}

// Check current session (project-based)
GET /api/auth/local/session?projectId={projectId}
Authorization: Bearer {token}

// Response
{
  "user": {
    "id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "project": {
      "id": "507f1f77bcf86cd799439011",
      "name": "My Awesome App",
      "role": "developer"
    }
  },
  "authenticated": true
}

// Response
{
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "john.doe@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "org": {
      "_id": "507f1f77bcf86cd799439013",
      "name": "Acme Corp"
    }
  }
}

// Refresh token (if implemented)
POST /api/auth/refresh
Authorization: Bearer {token}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": "24h"
}

// Logout (organization-based)
POST /api/auth/logout
Authorization: Bearer {token}

// Response
{
  "message": "Logout successful"
}

// Logout (project-based)
POST /api/auth/local/logout
Authorization: Bearer {token}

// Response
{
  "message": "Logout successful"
}
```

**Key Differences Between Organization-based and Project-based Auth:**

- **Organization-based** (`/api/auth/*`): Creates organizations automatically, simpler flow, good for admin/owner accounts
- **Project-based** (`/api/auth/local/*`): Requires existing project, project-specific authentication, includes rate limiting and captcha verification, better for end-user authentication

### 3. OAuth Authentication

#### Supported Providers
- Google, GitHub, Facebook, Microsoft, Apple, LinkedIn, Discord, Twitter, etc.

#### OAuth Flow Example (Google)

```javascript
// Step 1: Initiate OAuth
// Redirect user to:
GET /api/auth/oauth/google?projectId=507f1f77bcf86cd799439011

// User is redirected to Google consent screen
// After consent, Google redirects to:
GET /api/auth/oauth/google/callback?code={authorization_code}

// Step 2: Backend processes OAuth callback
// Backend automatically:
// 1. Exchanges code for access token
// 2. Fetches user profile from Google
// 3. Creates/updates user in database
// 4. Generates JWT token
// 5. Redirects to frontend with token

// Frontend receives redirect:
https://yourapp.com/auth/callback?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// Step 3: Use token for authenticated requests
Authorization: Bearer {token}

// Response (from OAuth callback redirect)
// Frontend receives:
https://yourapp.com/auth/callback?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...&user={"id":"507f1f77bcf86cd799439012","email":"john.doe@example.com"}
```

#### OAuth Provider Configuration

```javascript
// Configure OAuth provider in project settings
PATCH /api/projects/{projectId}
Authorization: Bearer {admin_token}
Content-Type: application/json

{
  "authProviders": {
    "google": {
      "enabled": true,
      "clientId": "your-google-client-id",
      "clientSecret": "your-google-client-secret",
      "callbackUrl": "https://api.mudbase.com/api/auth/oauth/google/callback"
    },
    "github": {
      "enabled": true,
      "clientId": "your-github-client-id",
      "clientSecret": "your-github-client-secret"
    }
  }
}
```

### 4. Magic Link Authentication

```javascript
// Step 1: Request Magic Link
POST /api/auth/magic-link/send
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "projectId": "507f1f77bcf86cd799439011",
  "redirectUrl": "https://yourapp.com/auth/verify"
}

// Response
{
  "success": true,
  "message": "Magic link sent to email"
}

// Step 2: User clicks link in email
// Email contains: https://api.mudbase.com/api/auth/magic-link/verify?token={magic_token}

// Step 3: Verify Magic Link
GET /api/auth/magic-link/verify?token={magic_token}&projectId={projectId}

// Response (redirects to frontend with token)
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

### 5. OTP Authentication (SMS/Email)

```javascript
// Step 1: Request OTP
POST /api/auth/otp/send
Content-Type: application/json

{
  "identifier": "john.doe@example.com", // or phone number
  "method": "email", // or "sms" or "auto"
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "message": "OTP sent to email",
  "expiresIn": 300 // 5 minutes
}

// Step 2: Verify OTP
POST /api/auth/otp/verify
Content-Type: application/json

{
  "identifier": "john.doe@example.com",
  "otp": "123456",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

### 6. Two-Factor Authentication (2FA)

```javascript
// Step 1: Enable 2FA
POST /api/auth/2fa/setup
Authorization: Bearer {token}
Content-Type: application/json

// Response
{
  "secret": "JBSWY3DPEHPK3PXP",
  "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..."
}

// Step 2: Verify and Enable
POST /api/auth/2fa/verify
Authorization: Bearer {token}
Content-Type: application/json

{
  "token": "123456" // TOTP code from authenticator app
}

// Step 3: Login with 2FA
POST /api/auth/local/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "[USER_PASSWORD]",
  "projectId": "507f1f77bcf86cd799439011"
}

// Response (requires 2FA)
{
  "requires2FA": true,
  "tempToken": "temp-token-for-2fa-verification"
}

// Step 4: Verify 2FA
POST /api/auth/2fa/verify-login
Content-Type: application/json

{
  "tempToken": "temp-token-for-2fa-verification",
  "token": "123456"
}

// Response
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

---

## Organization & Project Setup

### Complete Setup Workflow

```javascript
// Step 1: Create Organization
POST /api/orgs
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Acme Corporation",
  "slug": "acme-corp" // auto-generated if not provided
}

// Response
{
  "success": true,
  "org": {
    "_id": "507f1f77bcf86cd799439013",
    "name": "Acme Corporation",
    "slug": "acme-corp",
    "members": ["507f1f77bcf86cd799439012"],
    "createdAt": "2024-01-15T10:00:00Z"
  }
}

// Step 2: Create Project
POST /api/projects
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "My Awesome App",
  "orgId": "507f1f77bcf86cd799439013",
  "description": "A revolutionary app"
}

// Response
{
  "success": true,
  "project": {
    "_id": "507f1f77bcf86cd799439011",
    "name": "My Awesome App",
    "slug": "my-awesome-app",
    "org": "507f1f77bcf86cd799439013",
    "apiKey": "pk_live_abc123...",
    // NOTE: secretKey is NEVER returned in responses
    "settings": {
      "auth": {
        "requireEmailVerification": true,
        "sessionTimeout": 1800000
      }
    }
  }
}

// Step 3: Configure Project Settings
PATCH /api/projects/507f1f77bcf86cd799439011
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "settings": {
    "auth": {
      "providers": {
        "local": { "enabled": true },
        "google": { "enabled": true },
        "github": { "enabled": true },
        "magicLink": { "enabled": true },
        "otp": { "enabled": true }
      },
      "requireEmailVerification": true,
      "sessionTimeout": 1800000,
      "passwordPolicy": {
        "minLength": 8,
        "requireUppercase": true,
        "requireNumbers": true,
        "requireSpecialChars": true
      }
    },
    "database": {
      "collections": {
        "users": { "enabled": true },
        "products": { "enabled": true },
        "orders": { "enabled": true }
      }
    },
    "storage": {
      "maxFileSize": 10485760, // 10MB
      "allowedTypes": ["image/jpeg", "image/png", "application/pdf"]
    }
  }
}

// Step 4: Set up Payment Gateway (for billing)
POST /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "provider": "paystack",
  "publicKey": "pk_test_abc123...",
  "secretKey": "[SECRET_KEY_NOT_RETURNED]", // Only sent during creation, never returned
  "config": {
    "currency": "NGN",
    "webhookUrl": "https://api.mudbase.com/api/billing/webhooks/paystack"
  }
}

// Response
{
  "message": "Payment gateway created successfully",
  "gateway": {
    "_id": "507f1f77bcf86cd799439014",
    "provider": "paystack",
    "status": "pending"
    // NOTE: secretKey and webhookSecret are NEVER returned in responses
  }
}

// Step 5: Activate Payment Gateway
PATCH /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "status": "active"
}
```

---

## Wallet as a Service Workflows

### Important Security Notes

1. **Private Key Generation**: Each wallet gets its own unique private key. Keys are generated per wallet, not shared per project or organization.
2. **Private Key Visibility**: Private keys are ONLY shown once in the `/api/wallet/generate-key` endpoint response. They are NEVER returned in:
   - Wallet creation responses
   - Wallet listing responses
   - Wallet balance responses
   - Any other wallet-related endpoints
3. **Encryption**: All private keys are encrypted using AES-256-GCM before storage in the database.
4. **Key Storage**: Users must save the private key securely when generated. It cannot be retrieved later.

### Understanding Wallet Encryption

**Two Types of Keys**:

1. **Project Encryption Key** (`walletEncryptionKey`):
   - One per project (auto-generated)
   - Used to encrypt/decrypt wallet private keys for database storage
   - Provides additional security layer and project isolation
   - **NOT used for signing transactions**

2. **Wallet Private Keys**:
   - One unique key per wallet (generated per wallet)
   - The actual cryptocurrency private key
   - Used to sign blockchain transactions
   - Encrypted using project encryption key before storage

**Transaction Flow**:
- When making a transaction, the wallet's private key is decrypted using the project encryption key
- The decrypted wallet private key is then used to sign the transaction
- The project encryption key is NEVER used to sign transactions - only the wallet's private key

For detailed explanation, see `docs/WALLET_ENCRYPTION_EXPLAINED.md`.

### Supported Cryptocurrencies
- **BTC** (Bitcoin) - Bech32 addresses (bc1...)
- **ETH** (Ethereum)
- **BNB** (Binance Smart Chain)
- **SOL** (Solana)
- **TRX** (Tron)
- **LTC** (Litecoin)
- **USDT** (Tether - Multi-network: ETH, TRX, BSC, SOL, POLYGON)

### Platform Fee Structure (Hybrid Model)

**Formula**: `Platform Fee = max(1% * amount, minimumFee)`

| Currency | Minimum Fee | Minimum Withdrawal | Notes |
|----------|-------------|-------------------|-------|
| BTC | 0.00012 BTC (~$8) | 0.0002 BTC (~$13) | Updated based on market rates |
| ETH | 0.001 ETH (~$2.50) | 0.001 ETH (~$2.50) | Increased to protect against gas spikes |
| BNB | 0.0002 BNB (~$0.13) | 0.001 BNB (~$0.65) | Updated based on market rates |
| SOL | 0.008 SOL (~$1.20) | 0.01 SOL (~$1.50) | Platform as feePayer (no pre-fund) |
| TRX | 1 TRX (~$0.05) | 5 TRX (~$0.25) | Lowered for micro-transactions support |
| LTC | 0.0001 LTC (~$0.01) | 0.001 LTC (~$0.10) | Updated based on market rates |
| USDT-ETH | $4 fixed | 5 USDT | Covers gas spikes |
| USDT-BSC | $0.75 fixed | 2 USDT | Scaled with network cost + profit margin |
| USDT-TRX | $0.50 fixed | 2 USDT | Scaled with network cost + profit margin |
| USDT-SOL | $0.15 fixed | 2 USDT | Scaled with network cost + profit margin |

**Profitability Guardrails**: Transactions are rejected if platform fee < estimated network cost + pre-fund cost.

**Important Notes**:
- **ETH Gas Spikes**: ETH minimum fee (0.001 ETH) protects against gas spikes, but network conditions may still cause fees to exceed minimum during extreme congestion. Consider implementing dynamic fee adjustments for ETH during high gas periods.
- **USDT Fees**: Fees on non-ETH chains (BSC, TRX, SOL) are scaled with network cost + profit margin rather than flat rates for better user experience and competitiveness.
- **Micro-Transactions**: TRX minimum withdrawal lowered to 5 TRX to support small users and micro-transactions.

For detailed fee structure and rationale, see [FEE_STRUCTURE_UPDATED.md](./FEE_STRUCTURE_UPDATED.md)

### 1. Generate Private Key (Dashboard Endpoint)

**Important**: Each wallet gets its own unique private key. Keys are generated per wallet, not per project. The private key is only shown once in this response and must be saved securely.

```javascript
// Generate a new key pair for any supported currency
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "network": null // Only required for USDT
}

// Response
{
  "success": true,
  "data": {
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "privateKey": "L5EZftvrYaSudiozVRzTqLcHLNDoVn7H5HSfM9BAN6pMA5zoi3WZ",
    "publicKey": "02a1633cafcc01ebfb6d78e39f687a1f0995c62fc95f51ead10a02ee0be551b5fb"
  },
  "warning": "This private key is shown only once. Store it securely."
}

// For USDT with network
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "USDT",
  "network": "ETH" // Options: ETH, TRX, BSC, SOL, POLYGON
}

// Response
{
  "success": true,
  "data": {
    "currency": "USDT",
    "network": "ETH",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "privateKey": "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
    "publicKey": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
  },
  "warning": "This private key is shown only once. Store it securely."
}
```

### 2. Create Wallet with Generated Private Key

**Security Note**: Private keys are NEVER returned in wallet creation responses. They are encrypted and stored securely. Only the address and wallet metadata are returned.

```javascript
// User can set the generated private key from dashboard
POST /api/wallet/create
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "privateKey": "[PRIVATE_KEY_FROM_GENERATE_KEY_ENDPOINT]",
  "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh"
}

// Response
{
  "success": true,
  "message": "BTC wallet created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439015",
    "user": "507f1f77bcf86cd799439012",
    "org": "507f1f77bcf86cd799439013",
    "project": "507f1f77bcf86cd799439011",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "balance": "0",
    "isCustomKey": true,
    "isActive": true,
    "createdAt": "2024-01-15T10:00:00Z",
    "updatedAt": "2024-01-15T10:00:00Z"
  }
}
```

### 3. Create Wallet with Auto-Generated Key

```javascript
// Let system generate the key pair
POST /api/wallet/create
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "ETH"
}

// Response
{
  "success": true,
  "message": "ETH wallet created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439016",
    "user": "507f1f77bcf86cd799439012",
    "org": "507f1f77bcf86cd799439013",
    "project": "507f1f77bcf86cd799439011",
    "currency": "ETH",
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "balance": "0",
    "isCustomKey": false,
    "isActive": true,
    "createdAt": "2024-01-15T10:00:00Z",
    "updatedAt": "2024-01-15T10:00:00Z"
  }
}
```

### 4. Get User Wallets

```javascript
// Get all wallets for authenticated user
GET /api/wallet
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": [
    {
      "_id": "507f1f77bcf86cd799439015",
      "currency": "BTC",
      "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
      "balance": "0.05",
      "isCustomKey": true,
      "isActive": true,
      "project": "507f1f77bcf86cd799439011",
      "createdAt": "2024-01-15T10:00:00Z"
    },
    {
      "_id": "507f1f77bcf86cd799439016",
      "currency": "ETH",
      "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
      "balance": "2.5",
      "isCustomKey": false,
      "isActive": true,
      "project": "507f1f77bcf86cd799439011",
      "createdAt": "2024-01-15T10:00:00Z"
    }
  ]
}
```

### 4.5. Get Wallet Private Key

**Security Warning**: This endpoint returns the decrypted private key. Use only when you need to export your wallet. The private key is sensitive and should be kept secure.

```javascript
// Get private key for a specific wallet
GET /api/wallet/{walletId}/private-key
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "walletId": "507f1f77bcf86cd799439015",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "privateKey": "L5EZftvrYaSudiozVRzTqLcHLNDoVn7H5HSfM9BAN6pMA5zoi3WZ",
    "isCustomKey": true
  },
  "warning": "Keep this private key secure and never share it. Anyone with access to this key can control your wallet.",
  "security": {
    "accessedAt": "2024-01-15T11:20:00Z",
    "accessedBy": "507f1f77bcf86cd799439012"
  }
}
```

**Access Control**:
- User can only retrieve private keys for wallets they own
- Wallet must belong to the user's organization
- All access is logged for security audit

### 6. Get Wallet Balance

```javascript
// Get balance for specific wallet
GET /api/wallet/507f1f77bcf86cd799439015/balance
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "walletId": "507f1f77bcf86cd799439015",
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "balance": "0.05",
    "balanceInUSD": 2500.00,
    "lastSyncedAt": "2024-01-15T10:30:00Z"
  }
}
```

### 7. Withdraw Funds (Async Processing)

**Important**: Withdrawals are processed asynchronously. The API returns immediately with a `queued` status. You must check the transaction status to see when it's confirmed. Real blockchain confirmations take time (BTC: 10-60 minutes, ETH: 15-45 seconds, SOL: 5-10 seconds, etc.).

**Fee Model**: Platform fees use a hybrid model: `max(1% * amount, minimumFee)`. Project fees (optional) are added on top if configured. Minimum fees are enforced per currency to ensure profitability.

```javascript
// Withdraw from wallet
POST /api/wallet/507f1f77bcf86cd799439015/withdraw
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "toAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "amount": 0.01,
  "feeRate": 10 // For BTC/LTC only
}

// Response (Returns Immediately - Async Processing)
{
  "success": true,
  "message": "Withdrawal is processing",
  "data": {
    "transactionId": "507f1f77bcf86cd799439027",
    "status": "queued",
    "amount": 0.01,
    "platformFee": 0.00012, // max(1% * 0.01, 0.00012 BTC minimum)
    "projectFee": 0.00005, // Optional: if project fee is configured
    "totalFee": 0.00017, // Platform fee + project fee
    "message": "Withdrawal is processing. Check transaction status for updates."
  }
}

// Check Transaction Status
GET /api/wallet/transactions/507f1f77bcf86cd799439027
Authorization: Bearer {user_token}

// Response (During Processing)
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439027",
    "status": "prefunding", // or "processing", "broadcasted"
    "mainTxStatus": "pending",
    "mainTxHash": null,
    "platformFee": 0.0005,
    "createdAt": "2024-01-15T10:35:00Z"
  }
}

// Response (After Confirmation)
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439027",
    "status": "confirmed",
    "mainTxHash": "abc123def456ghi789jkl012mno345pqr678stu901vwx234yz",
    "mainTxStatus": "confirmed",
    "refundTxHash": "xyz789abc123def456ghi789jkl012mno345pqr678",
    "refundStatus": "confirmed",
    "networkFee": 0.0001,
    "platformFee": 0.0005,
    "completedAt": "2024-01-15T10:45:00Z"
  }
}
```

**Transaction Status Flow**:
- `queued` → Transaction added to processing queue
- `prefunding` → Platform sending pre-fund (if needed)
- `processing` → User transaction being prepared
- `broadcasted` → Transaction broadcast to blockchain
- `confirmed` → Transaction confirmed on blockchain
- `completed` → All operations completed successfully
- `failed` → Transaction failed (check error field)
- `partial` → User tx succeeded but refund failed (CRITICAL - requires manual intervention)

### 8. Validate Address

```javascript
// Validate cryptocurrency address
POST /api/wallet/validate-address
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "currency": "BTC"
}

// Response
{
  "success": true,
  "data": {
    "isValid": true,
    "currency": "BTC",
    "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
    "format": "bech32"
  }
}
```

### 9. Get Supported Currencies

```javascript
// Get list of supported currencies
GET /api/wallet/currencies
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": [
    {
      "code": "BTC",
      "name": "Bitcoin",
      "network": "mainnet"
    },
    {
      "code": "ETH",
      "name": "Ethereum",
      "network": "mainnet"
    },
    {
      "code": "BNB",
      "name": "Binance Coin",
      "network": "bsc"
    },
    {
      "code": "LTC",
      "name": "Litecoin",
      "network": "mainnet"
    },
    {
      "code": "SOL",
      "name": "Solana",
      "network": "mainnet"
    },
    {
      "code": "TRX",
      "name": "Tron",
      "network": "mainnet"
    },
    {
      "code": "USDT",
      "name": "Tether",
      "network": "ethereum"
    }
  ]
}
```

---

## Billing & Payment Gateway Workflows

### 1. Setup Payment Gateway

```javascript
// Create payment gateway account
POST /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "provider": "paystack", // or "flutterwave", "monnify", "interswitch"
  "publicKey": "pk_test_abc123...",
  "secretKey": "[SECRET_KEY_NOT_RETURNED]",
  "config": {
    "currency": "NGN",
    "webhookUrl": "https://api.mudbase.com/api/billing/webhooks/paystack"
  }
}

// Response
{
  "message": "Payment gateway created successfully",
  "gateway": {
    "_id": "507f1f77bcf86cd799439017",
    "provider": "paystack",
    "status": "pending",
    "publicKey": "pk_test_abc123...",
    "createdAt": "2024-01-15T10:40:00Z"
  }
}

// Activate gateway
PATCH /api/billing/orgs/507f1f77bcf86cd799439013/payment-gateway
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "status": "active"
}
```

### 2. Create Subscription Plan

```javascript
// Create a plan for your project
POST /api/billing/projects/507f1f77bcf86cd799439011/plans
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Pro Plan",
  "description": "Professional plan with advanced features",
  "pricing": {
    "type": "recurring",
    "monthly": {
      "amount": 5000, // in kobo (50 NGN)
      "currency": "NGN"
    },
    "yearly": {
      "amount": 50000, // in kobo (500 NGN)
      "currency": "NGN"
    },
    "usage": [
      {
        "metric": "api_calls",
        "unitPrice": 0.1, // per API call
        "includedUnits": 10000,
        "currency": "NGN"
      }
    ]
  },
  "features": [
    {
      "name": "api_access",
      "description": "Full API access",
      "included": true
    },
    {
      "name": "storage",
      "description": "100GB storage",
      "included": true,
      "limit": 107374182400 // 100GB in bytes
    }
  ],
  "limits": {
    "apiCalls": 100000,
    "storage": 107374182400,
    "bandwidth": 1073741824000
  },
  "trial": {
    "enabled": true,
    "days": 7
  },
  "isActive": true,
  "isDefault": false
}

// Response
{
  "message": "Plan created successfully",
  "plan": {
    "_id": "507f1f77bcf86cd799439018",
    "name": "Pro Plan",
    "description": "Professional plan with advanced features",
    "pricing": {
      "type": "recurring",
      "monthly": {
        "amount": 5000,
        "currency": "NGN"
      },
      "yearly": {
        "amount": 50000,
        "currency": "NGN"
      }
    },
    "features": [
      {
        "name": "api_access",
        "description": "Full API access",
        "included": true
      },
      {
        "name": "storage",
        "description": "100GB storage",
        "included": true,
        "limit": 107374182400
      }
    ],
    "isActive": true,
    "createdAt": "2024-01-15T10:40:00Z"
  }
}
```

### 3. Customer Checkout Flow

```javascript
// Step 1: Get available plans (Public API)
GET /api/billing/public/projects/507f1f77bcf86cd799439011/plans

// Response
{
  "plans": [
    {
      "_id": "507f1f77bcf86cd799439018",
      "name": "Pro Plan",
      "description": "Professional plan with advanced features",
      "pricing": {
        "monthly": {
          "amount": 5000,
          "currency": "NGN"
        },
        "yearly": {
          "amount": 50000,
          "currency": "NGN"
        }
      },
      "features": [ ... ]
    }
  ]
}

// Step 2: Create checkout session
POST /api/billing/public/projects/507f1f77bcf86cd799439011/checkout
Content-Type: application/json

{
  "planId": "507f1f77bcf86cd799439018",
  "billingCycle": "monthly",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  },
  "successUrl": "https://yourapp.com/billing/success",
  "cancelUrl": "https://yourapp.com/billing/cancel"
}

// Response
{
  "success": true,
  "data": {
    "checkoutUrl": "https://paystack.com/pay/abc123...",
    "reference": "mudbase_1705315200_abc123def456",
    "amount": 5000,
    "currency": "NGN"
  }
}

// Step 3: Redirect customer to authorizationUrl
// Customer completes payment on Paystack/Flutterwave

// Step 4: Payment Gateway redirects to callback URL
// Backend automatically verifies payment and creates subscription

// Step 5: Verify payment (if needed manually)
POST /api/billing/public/projects/507f1f77bcf86cd799439011/verify-payment
Content-Type: application/json

{
  "reference": "mudbase_1705315200_abc123def456",
  "orgId": "507f1f77bcf86cd799439013"
}

// Response
{
  "success": true,
  "message": "Payment verified and subscription created",
  "data": {
    "subscription": {
      "_id": "507f1f77bcf86cd799439019",
      "status": "active",
      "plan": {
        "_id": "507f1f77bcf86cd799439018",
        "name": "Pro Plan"
      },
      "customerEmail": "customer@example.com",
      "currentPeriodEnd": "2024-02-15T10:45:00Z",
      "billingCycle": "monthly"
    }
  }
}
```

### 4. Check Subscription Status

```javascript
// Check customer subscription (Public API)
GET /api/billing/public/projects/507f1f77bcf86cd799439011/subscription?email=customer@example.com

// Response
{
  "hasSubscription": true,
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active",
    "plan": {
      "_id": "507f1f77bcf86cd799439018",
      "name": "Pro Plan",
      "price": 5000,
      "currency": "NGN"
    },
    "customerEmail": "customer@example.com",
    "currentPeriodEnd": "2024-02-15T10:45:00Z",
    "billingCycle": "monthly",
    "createdAt": "2024-01-15T10:45:00Z"
  }
}
```

### 5. Check Feature Access

```javascript
// Check if customer has access to a feature
GET /api/billing/public/projects/507f1f77bcf86cd799439011/feature-access?email=customer@example.com&feature=api_access

// Response
{
  "hasAccess": true,
  "reason": "Active subscription",
  "plan": {
    "_id": "507f1f77bcf86cd799439018",
    "name": "Pro Plan"
  },
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active"
  }
}
```

### 6. Record Usage (Metered Billing)

```javascript
// Record usage for metered billing
POST /api/billing/public/projects/507f1f77bcf86cd799439011/usage
Content-Type: application/json

{
  "email": "customer@example.com",
  "metric": "api_calls",
  "quantity": 150
}

// Response
{
  "success": true,
  "message": "Usage recorded successfully"
}
```

### 7. Cancel Subscription

```javascript
// Cancel subscription
POST /api/billing/subscriptions/507f1f77bcf86cd799439019/cancel
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "cancelImmediately": false // Cancel at period end
}

// Response
{
  "message": "Subscription canceled successfully",
  "subscription": {
    "_id": "507f1f77bcf86cd799439019",
    "status": "active",
    "cancelAtPeriodEnd": true,
    "currentPeriodEnd": "2024-02-15T10:45:00Z",
    "canceledAt": "2024-01-15T11:00:00Z"
  }
}
```

### 8. Payment Gateway Webhook

```javascript
// Payment gateway sends webhook on payment events
POST /api/billing/webhooks/paystack
Content-Type: application/json
X-Paystack-Signature: {signature}

{
  "event": "charge.success",
  "data": {
    "reference": "mudbase_1705315200_abc123def456",
    "amount": 5000,
    "currency": "NGN",
    "customer": {
      "email": "customer@example.com"
    },
    "metadata": {
      "projectId": "507f1f77bcf86cd799439011",
      "planId": "507f1f77bcf86cd799439018",
      "billingCycle": "monthly",
      "orgId": "507f1f77bcf86cd799439013"
    }
  }
}

// Backend automatically:
// 1. Verifies webhook signature
// 2. Verifies payment
// 3. Creates subscription
// 4. Sends confirmation email
// 5. Triggers project webhook
```

---

## Database Operations

### 1. Create Collection Schema

```javascript
// Define collection schema
POST /api/projects/507f1f77bcf86cd799439011/schemas
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "products",
  "fields": [
    {
      "name": "name",
      "type": "string",
      "required": true,
      "indexed": true
    },
    {
      "name": "price",
      "type": "number",
      "required": true
    },
    {
      "name": "description",
      "type": "string"
    },
    {
      "name": "category",
      "type": "string",
      "indexed": true
    },
    {
      "name": "inStock",
      "type": "boolean",
      "default": true
    },
    {
      "name": "createdAt",
      "type": "date",
      "default": "now"
    }
  ],
  "indexes": [
    {
      "fields": ["category", "price"],
      "unique": false
    }
  ]
}

// Response
{
  "success": true,
  "collection": {
    "_id": "507f1f77bcf86cd799439020",
    "name": "products",
    "slug": "products",
    "project": "507f1f77bcf86cd799439011",
    "fields": [
      {
        "name": "name",
        "type": "string",
        "required": true,
        "indexed": true
      },
      {
        "name": "price",
        "type": "number",
        "required": true
      },
      {
        "name": "description",
        "type": "string"
      },
      {
        "name": "category",
        "type": "string",
        "indexed": true
      },
      {
        "name": "inStock",
        "type": "boolean",
        "default": true
      }
    ],
    "createdAt": "2024-01-15T10:50:00Z"
  }
}
```

### 2. Create Document

```javascript
// Create a document in collection
POST /api/projects/507f1f77bcf86cd799439011/data/products
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Laptop",
  "price": 999.99,
  "description": "High-performance laptop",
  "category": "electronics",
  "inStock": true
}

// Response
{
  "message": "Data created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439021",
    "name": "Laptop",
    "price": 999.99,
    "description": "High-performance laptop",
    "category": "electronics",
    "inStock": true,
    "createdAt": "2024-01-15T10:55:00Z",
    "updatedAt": "2024-01-15T10:55:00Z"
  }
}
```

### 3. Query Documents

```javascript
// Get all documents with filters
GET /api/projects/507f1f77bcf86cd799439011/data/products?category=electronics&price[gte]=500&limit=10&page=1
Authorization: Bearer {user_token}

// Response
{
  "data": [
    {
      "_id": "507f1f77bcf86cd799439021",
      "name": "Laptop",
      "price": 999.99,
      "description": "High-performance laptop",
      "category": "electronics",
      "inStock": true,
      "createdAt": "2024-01-15T10:55:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 1,
    "totalPages": 1
  }
}

// Advanced query
POST /api/projects/507f1f77bcf86cd799439011/data/products/query
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "filter": {
    "$and": [
      { "category": "electronics" },
      { "price": { "$gte": 500 } },
      { "inStock": true }
    ]
  },
  "sort": { "price": -1 },
  "limit": 10,
  "skip": 0
}
```

### 4. Update Document

```javascript
// Update document
PATCH /api/projects/507f1f77bcf86cd799439011/data/products/507f1f77bcf86cd799439021
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "price": 899.99,
  "inStock": false
}

// Response
{
  "message": "Data updated successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439021",
    "name": "Laptop",
    "price": 899.99,
    "description": "High-performance laptop",
    "category": "electronics",
    "inStock": false,
    "createdAt": "2024-01-15T10:55:00Z",
    "updatedAt": "2024-01-15T11:00:00Z"
  }
}
```

### 5. Delete Document

```javascript
// Delete document
DELETE /api/projects/507f1f77bcf86cd799439011/data/products/507f1f77bcf86cd799439021
Authorization: Bearer {user_token}

// Response
{
  "message": "Data deleted successfully"
}
```

---

## Real-time Chat System

### 1. Create Chat

```javascript
// Create a new chat
POST /api/projects/507f1f77bcf86cd799439011/chats
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Team Discussion",
  "type": "group", // or "direct"
  "participants": [
    "507f1f77bcf86cd799439012",
    "507f1f77bcf86cd799439022"
  ],
  "metadata": {
    "projectId": "507f1f77bcf86cd799439011"
  }
}

// Response
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439023",
    "name": "Team Discussion",
    "type": "group",
    "participants": [
      {
        "user": "507f1f77bcf86cd799439012",
        "role": "admin",
        "joinedAt": "2024-01-15T11:05:00Z"
      },
      {
        "user": "507f1f77bcf86cd799439022",
        "role": "member",
        "joinedAt": "2024-01-15T11:05:00Z"
      }
    ],
    "createdBy": "507f1f77bcf86cd799439012",
    "project": "507f1f77bcf86cd799439011",
    "createdAt": "2024-01-15T11:05:00Z"
  }
}
```

### 2. Send Message (HTTP)

```javascript
// Send message via HTTP
POST /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "content": "Hello team! How's everyone doing?",
  "type": "text" // or "image", "file", etc.
}

// Response
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439024",
    "content": "Hello team! How's everyone doing?",
    "type": "text",
    "sender": "507f1f77bcf86cd799439012",
    "chat": "507f1f77bcf86cd799439023",
    "project": "507f1f77bcf86cd799439011",
    "createdAt": "2024-01-15T11:06:00Z",
    "updatedAt": "2024-01-15T11:06:00Z"
  }
}
```

### 3. Real-time Messaging (Socket.IO)

```javascript
// Client-side Socket.IO connection
import io from 'socket.io-client';

const socket = io('https://api.mudbase.com', {
  auth: {
    token: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
  }
});

// Join chat room
socket.emit('chat:join', {
  chatId: '507f1f77bcf86cd799439023',
  projectId: '507f1f77bcf86cd799439011'
});

// Send message via Socket.IO
socket.emit('chat:message:send', {
  chatId: '507f1f77bcf86cd799439023',
  content: 'Hello from real-time!',
  type: 'text'
});

// Listen for new messages
socket.on('chat:message:new', (data) => {
  console.log('New message:', data);
  // {
  //   message: {
  //     _id: '507f1f77bcf86cd799439025',
  //     content: 'Hello from real-time!',
  //     sender: { ... },
  //     createdAt: '2024-01-15T11:07:00Z'
  //   },
  //   chatId: '507f1f77bcf86cd799439023'
  // }
});

// Typing indicator
socket.emit('chat:typing', {
  chatId: '507f1f77bcf86cd799439023',
  isTyping: true
});

socket.on('chat:typing', (data) => {
  console.log('User typing:', data);
  // {
  //   userId: '507f1f77bcf86cd799439012',
  //   chatId: '507f1f77bcf86cd799439023',
  //   isTyping: true
  // }
});

// Voice/Video call events
socket.emit('chat:call:initiate', {
  chatId: '507f1f77bcf86cd799439023',
  type: 'video' // or 'voice'
});

socket.on('chat:call:incoming', (data) => {
  console.log('Incoming call:', data);
});

socket.emit('chat:call:accept', {
  callId: 'call_abc123'
});

socket.emit('chat:call:reject', {
  callId: 'call_abc123'
});

socket.emit('chat:call:end', {
  callId: 'call_abc123'
});
```

### 4. Edit Message

```javascript
// Edit message
PATCH /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "content": "Hello team! How's everyone doing? (edited)"
}

// Socket.IO event also emitted: 'chat:message:updated'
```

### 5. Delete Message

```javascript
// Delete message
DELETE /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024
Authorization: Bearer {user_token}

// Socket.IO event also emitted: 'chat:message:deleted'
```

### 6. Add Reaction

```javascript
// Add reaction to message
POST /api/projects/507f1f77bcf86cd799439011/chats/507f1f77bcf86cd799439023/messages/507f1f77bcf86cd799439024/reactions
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "emoji": "👍"
}

// Socket.IO event also emitted: 'chat:message:reaction'
```

---

## Integration System

### 1. Create Integration

```javascript
// Create integration
POST /api/projects/507f1f77bcf86cd799439011/integrations
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "name": "Twitter API Integration",
  "provider": "twitter",
  "category": "social",
    "settings": {
      "isActive": true
      // NOTE: API keys, secrets, and tokens are encrypted and never returned in responses
    },
  "config": {
    "rateLimit": 100,
    "timeout": 5000
  }
}

// Response
{
  "integration": {
    "_id": "507f1f77bcf86cd799439026",
    "name": "Twitter API Integration",
    "provider": "twitter",
    "category": "social",
    "project": "507f1f77bcf86cd799439011",
    "settings": {
      "isActive": true
    },
    "createdAt": "2024-01-15T11:10:00Z",
    "updatedAt": "2024-01-15T11:10:00Z"
  }
}
```

### 2. Execute Integration

```javascript
// Execute integration
POST /api/projects/507f1f77bcf86cd799439011/integrations/507f1f77bcf86cd799439026/execute
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "method": "POST",
  "endpoint": "/2/tweets",
  "body": {
    "text": "Hello from MUDBASE!"
  },
  "headers": {
    "Content-Type": "application/json"
  }
}

// Response
{
  "success": true,
  "data": {
    "status": 201,
    "data": {
      "id": "1234567890",
      "text": "Hello from MUDBASE!",
      "created_at": "2024-01-15T11:12:00Z"
    },
    "headers": {
      "content-type": "application/json",
      "x-rate-limit-remaining": "299"
    }
  },
  "usage": {
    "apiCalls": 1,
    "timestamp": "2024-01-15T11:12:00Z"
  }
}
```

### 3. Get Integration Usage Stats

```javascript
// Get usage statistics
GET /api/projects/507f1f77bcf86cd799439011/integrations/507f1f77bcf86cd799439026/usage?period=month
Authorization: Bearer {user_token}

// Response
{
  "stats": {
    "totalCalls": 1250,
    "successfulCalls": 1200,
    "failedCalls": 50,
    "successRate": 96.0,
    "period": "month",
    "breakdown": [
      {
        "date": "2024-01-15",
        "calls": 45,
        "successful": 43,
        "failed": 2
      }
    ]
  }
}
```

---

## Complete End-to-End Scenarios

### Scenario 1: E-commerce Platform Setup

```javascript
// Step 1: Register Admin User
POST /api/auth/local/register
{
  "email": "admin@ecommerce.com",
  "password": "SecurePass123!",
  "firstName": "Admin",
  "lastName": "User"
}

// Step 2: Create Organization
POST /api/orgs
Authorization: Bearer {admin_token}
{
  "name": "Ecommerce Corp"
}

// Step 3: Create Project
POST /api/projects
Authorization: Bearer {admin_token}
{
  "name": "Ecommerce Platform",
  "orgId": "{org_id}"
}

// Step 4: Setup Payment Gateway
POST /api/billing/orgs/{org_id}/payment-gateway
Authorization: Bearer {admin_token}
{
  "provider": "paystack",
  "publicKey": "pk_test_...",
  "secretKey": "sk_test_..."
}

// Step 5: Create Product Schema
POST /api/projects/{project_id}/schemas
Authorization: Bearer {admin_token}
{
  "name": "products",
  "fields": [
    { "name": "name", "type": "string", "required": true },
    { "name": "price", "type": "number", "required": true },
    { "name": "stock", "type": "number", "required": true }
  ]
}

// Step 6: Create Order Schema
POST /api/projects/{project_id}/schemas
Authorization: Bearer {admin_token}
{
  "name": "orders",
  "fields": [
    { "name": "customerId", "type": "string", "required": true },
    { "name": "products", "type": "array", "required": true },
    { "name": "total", "type": "number", "required": true },
    { "name": "status", "type": "string", "default": "pending" }
  ]
}

// Step 7: Customer Registration
POST /api/auth/local/register
{
  "email": "customer@example.com",
  "password": "CustomerPass123!",
  "firstName": "John",
  "lastName": "Customer",
  "projectId": "{project_id}"
}

// Step 8: Customer Creates Order
POST /api/projects/{project_id}/data/orders
Authorization: Bearer {customer_token}
{
  "customerId": "{customer_id}",
  "products": [
    { "productId": "{product_id}", "quantity": 2 }
  ],
  "total": 1999.98,
  "status": "pending"
}

// Step 9: Process Payment
POST /api/billing/public/projects/{project_id}/checkout
{
  "planId": "{plan_id}",
  "billingCycle": "one-time",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  },
  "amount": 1999.98
}

// Step 10: Update Order Status
PATCH /api/projects/{project_id}/data/orders/{order_id}
Authorization: Bearer {admin_token}
{
  "status": "paid"
}
```

### Scenario 2: Crypto Wallet Integration

```javascript
// Step 1: User generates Bitcoin wallet key
POST /api/wallet/generate-key
Authorization: Bearer {user_token}
{
  "currency": "BTC"
}

// Response includes privateKey (shown once only) that user must save securely

// Step 2: User creates wallet with generated key
POST /api/wallet/create
Authorization: Bearer {user_token}
{
  "currency": "BTC",
  "privateKey": "[PRIVATE_KEY_FROM_STEP_1]",
  "address": "[ADDRESS_FROM_STEP_1]"
}

// Response does NOT include privateKey - it's encrypted and stored securely

// Step 3: Check wallet balance
GET /api/wallet/{wallet_id}/balance
Authorization: Bearer {user_token}

// Step 4: Receive payment (external)
// Bitcoin sent to wallet address

// Step 5: Withdraw funds (async processing)
POST /api/wallet/{wallet_id}/withdraw
Authorization: Bearer {user_token}
{
  "toAddress": "bc1q...",
  "amount": 0.01,
  "feeRate": 10
}

// Response (returns immediately - async processing)
{
  "success": true,
  "data": {
    "transactionId": "...",
    "status": "queued",
    "platformFee": 0.0005 // max(1% * 0.01, 0.0005 BTC minimum)
  }
}

// Step 6: Check transaction status (poll until confirmed)
GET /api/wallet/transactions/{transaction_id}
Authorization: Bearer {user_token}

// Response (after confirmation - may take 10-60 minutes for BTC)
{
  "success": true,
  "data": {
    "status": "confirmed",
    "mainTxHash": "...",
    "mainTxStatus": "confirmed"
  }
}
```

### Scenario 3: SaaS Subscription Flow

```javascript
// Step 1: Setup billing plan
POST /api/billing/projects/{project_id}/plans
Authorization: Bearer {admin_token}
{
  "name": "Premium",
  "pricing": {
    "monthly": { "amount": 10000, "currency": "NGN" },
    "yearly": { "amount": 100000, "currency": "NGN" }
  },
  "features": [
    { "name": "api_access", "included": true },
    { "name": "storage", "included": true, "limit": 107374182400 }
  ]
}

// Step 2: Customer subscribes
POST /api/billing/public/projects/{project_id}/checkout
{
  "planId": "{plan_id}",
  "billingCycle": "monthly",
  "customerInfo": {
    "email": "customer@example.com",
    "name": "John Customer"
  }
}

// Step 3: Check subscription status
GET /api/billing/public/projects/{project_id}/subscription?email=customer@example.com

// Step 4: Check feature access
GET /api/billing/public/projects/{project_id}/feature-access?email=customer@example.com&feature=api_access

// Step 5: Record usage
POST /api/billing/public/projects/{project_id}/usage
{
  "email": "customer@example.com",
  "metric": "api_calls",
  "quantity": 150
}
```

---

## API Reference Quick Guide

### Authentication Endpoints

```
POST   /api/auth/register                - Register new user (org-based)
POST   /api/auth/login                   - Login user (org-based)
POST   /api/auth/logout                  - Logout user (org-based)
GET    /api/auth/session                 - Get current session (org-based)
POST   /api/auth/password-reset          - Request password reset (org-based)
POST   /api/auth/password-reset/{token}  - Reset password (org-based)
POST   /api/auth/local/register          - Register new user (project-based)
POST   /api/auth/local/login             - Login user (project-based)
POST   /api/auth/local/logout            - Logout user (project-based)
GET    /api/auth/local/session           - Get current session (project-based)
POST   /api/auth/local/password-reset    - Request password reset (project-based)
POST   /api/auth/local/password-reset/{token} - Reset password (project-based)
GET    /api/auth/oauth/{provider}        - Initiate OAuth
GET    /api/auth/oauth/{provider}/callback - OAuth callback
POST   /api/auth/magic-link/send         - Send magic link
GET    /api/auth/magic-link/verify        - Verify magic link
POST   /api/auth/otp/send                - Send OTP
POST   /api/auth/otp/verify              - Verify OTP
POST   /api/auth/2fa/setup               - Setup 2FA
POST   /api/auth/2fa/verify              - Verify 2FA
```

### Wallet Endpoints

```
GET    /api/wallet/currencies            - Get supported currencies
POST   /api/wallet/generate-key          - Generate key pair
POST   /api/wallet/validate-address      - Validate address
POST   /api/wallet/create                 - Create wallet
GET    /api/wallet                        - Get user wallets
GET    /api/wallet/{id}/balance          - Get wallet balance
GET    /api/wallet/{id}/private-key      - Get wallet private key (WARNING: Sensitive)
POST   /api/wallet/{id}/withdraw          - Withdraw funds
```

### Billing Endpoints

```
POST   /api/billing/orgs/{orgId}/payment-gateway - Create gateway
GET    /api/billing/orgs/{orgId}/payment-gateway - Get gateway status
POST   /api/billing/projects/{id}/plans         - Create plan
GET    /api/billing/projects/{id}/plans         - Get plans
GET    /api/billing/public/projects/{id}/plans  - Get public plans
POST   /api/billing/public/projects/{id}/checkout - Create checkout
POST   /api/billing/public/projects/{id}/verify-payment - Verify payment
GET    /api/billing/public/projects/{id}/subscription - Check subscription
GET    /api/billing/public/projects/{id}/feature-access - Check feature access
POST   /api/billing/public/projects/{id}/usage   - Record usage
POST   /api/billing/subscriptions/{id}/cancel   - Cancel subscription
POST   /api/billing/webhooks/{provider}         - Payment webhook
```

### Database Endpoints

```
POST   /api/projects/{id}/schemas         - Create schema
GET    /api/projects/{id}/schemas         - Get schemas
POST   /api/projects/{id}/data/{collection} - Create document
GET    /api/projects/{id}/data/{collection} - Query documents
PATCH  /api/projects/{id}/data/{collection}/{id} - Update document
DELETE /api/projects/{id}/data/{collection}/{id} - Delete document
POST   /api/projects/{id}/data/{collection}/query - Advanced query
```

### Chat Endpoints

```
POST   /api/projects/{id}/chats            - Create chat
GET    /api/projects/{id}/chats           - Get user chats
POST   /api/projects/{id}/chats/{id}/messages - Send message
GET    /api/projects/{id}/chats/{id}/messages - Get messages
PATCH  /api/projects/{id}/chats/{id}/messages/{id} - Edit message
DELETE /api/projects/{id}/chats/{id}/messages/{id} - Delete message
POST   /api/projects/{id}/chats/{id}/messages/{id}/reactions - Add reaction
```

### Integration Endpoints

```
GET    /api/projects/{id}/integrations/templates - Get templates
POST   /api/projects/{id}/integrations          - Create integration
GET    /api/projects/{id}/integrations          - Get integrations
POST   /api/projects/{id}/integrations/{id}/test - Test integration
POST   /api/projects/{id}/integrations/{id}/execute - Execute integration
GET    /api/projects/{id}/integrations/{id}/usage - Get usage stats
```

---

## Security Features

### 1. Session Management
- **Idle Timeout**: 30 minutes of inactivity
- **Rolling Sessions**: Session extends on activity
- **Secure Cookies**: HttpOnly, Secure in production

### 2. Access Control
- **RBAC**: Role-Based Access Control (owner, admin, developer, viewer)
- **Project-Level Access**: Users can only access their project's resources
- **Organization-Level Access**: Users can only access their org's resources

### 3. Encryption
- **Private Keys**: AES-256-GCM encryption at rest
- **API Credentials**: Encrypted in database
- **Password Hashing**: bcrypt with salt rounds 12

### 4. Rate Limiting
- **Global**: 100 requests per 15 minutes per IP
- **Authentication**: 5 login attempts per 15 minutes
- **API Keys**: Configurable per project

### 5. Audit Logging
- All authentication events logged
- All authorization failures logged
- All sensitive operations logged

---

## Best Practices

### 1. Authentication
- Always use HTTPS in production
- Store JWT tokens securely (httpOnly cookies recommended)
- Implement token refresh mechanism
- Use 2FA for sensitive accounts

### 2. Wallet Management
- Never log private keys
- Always validate addresses before transactions
- Use custom keys only when necessary
- Keep backup of private keys securely
- **Async Processing**: Withdrawals are processed asynchronously - check transaction status for updates
- **Real Confirmations**: System uses real blockchain confirmations (not fake delays)
- **Minimum Fees**: Enforced per currency to ensure profitability
- **Status Tracking**: Monitor transaction status through: `queued` → `prefunding` → `processing` → `broadcasted` → `confirmed` → `completed`

### 3. Billing
- Verify webhook signatures
- Handle payment failures gracefully
- Implement retry logic for failed payments
- Monitor subscription status

### 4. Database
- Use indexes for frequently queried fields
- Implement pagination for large datasets
- Validate input data
- Use transactions for critical operations

### 5. Real-time
- Handle connection failures
- Implement reconnection logic
- Use rooms for efficient message delivery
- Clean up on disconnect

---

## Error Handling

### Common Error Codes

```javascript
// Authentication Errors
401 - Unauthorized (Invalid token)
403 - Forbidden (Insufficient permissions)
429 - Too Many Requests (Rate limit exceeded)

// Validation Errors
400 - Bad Request (Invalid input)
422 - Unprocessable Entity (Validation failed)

// Resource Errors
404 - Not Found (Resource doesn't exist)
409 - Conflict (Resource already exists)

// Server Errors
500 - Internal Server Error
503 - Service Unavailable
```

### Error Response Format

```javascript
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "email": "Email is required",
      "password": "Password must be at least 8 characters"
    }
  }
}
```

---

## Project Fee Management

### Overview

Project owners can configure their own fees per currency, which are collected in addition to platform fees. These fees are accumulated and paid out via automated bi-weekly payouts.

### 1. Configure Project Fee Settings

```javascript
// Step 1: Create or update fee settings
POST /api/projects/507f1f77bcf86cd799439011/fee-settings
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC",
  "enabled": true,
  "feeAmount": 0.00005, // Must be < platform minimum fee (0.00012 BTC)
  "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "payoutThreshold": 0.001 // Minimum amount before payout
}

// Response
{
  "success": true,
  "message": "Fee settings updated successfully",
  "data": {
    "feeSettings": {
      "BTC": {
        "enabled": true,
        "feeAmount": 0.00005,
        "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "payoutThreshold": 0.001,
        "addressVerified": false
      }
    }
  }
}
```

### 2. Verify Payout Address

```javascript
// Step 1: Initiate address verification
POST /api/projects/507f1f77bcf86cd799439011/fee-settings/BTC/verify-address
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "message": "Test transaction sent. Please confirm receipt or provide transaction hash from the address.",
  "data": {
    "verificationStatus": "pending",
    "testTxHash": "abc123def456...",
    "testAmount": 0.00001,
    "instructions": "Either confirm you received the test transaction, or send a transaction FROM the payout address to prove ownership."
  }
}

// Step 2: Confirm verification
POST /api/projects/507f1f77bcf86cd799439011/fee-settings/BTC/confirm-verification
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "txHash": "xyz789abc123..." // Transaction hash proving ownership
}

// Response
{
  "success": true,
  "message": "Address verified successfully",
  "data": {
    "verified": true,
    "verifiedAt": "2024-01-15T12:00:00Z"
  }
}
```

### 3. Check Fee Balance

```javascript
// Get balance for specific currency
GET /api/projects/507f1f77bcf86cd799439011/fee-balances/BTC
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "currency": "BTC",
    "collectedAmount": 0.0005,
    "threshold": 0.001,
    "status": "accumulating",
    "nextScheduledPayoutDate": null,
    "lastPayoutDate": null,
    "totalPaidOut": 0,
    "totalCollected": 0.0005
  }
}

// Get all balances
GET /api/projects/507f1f77bcf86cd799439011/fee-balances
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "balances": [
      {
        "currency": "BTC",
        "collectedAmount": 0.0005,
        "threshold": 0.001,
        "status": "accumulating"
      },
      {
        "currency": "ETH",
        "collectedAmount": 0.002,
        "threshold": 0.001,
        "status": "ready",
        "nextScheduledPayoutDate": "2024-01-17T02:00:00Z"
      }
    ]
  }
}
```

### 4. View Payout History

```javascript
// Get payout history
GET /api/projects/507f1f77bcf86cd799439011/payout-history?limit=10&offset=0
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "payouts": [
      {
        "_id": "507f1f77bcf86cd799439030",
        "currency": "BTC",
        "grossAmount": 0.001,
        "networkFee": 0.0001,
        "netAmount": 0.0009,
        "toAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "txHash": "abc123def456...",
        "status": "completed",
        "scheduledAt": "2024-01-15T02:00:00Z",
        "processedAt": "2024-01-15T02:05:00Z",
        "confirmedAt": "2024-01-15T02:45:00Z",
        "createdAt": "2024-01-15T02:00:00Z"
      }
    ],
    "total": 1,
    "limit": 10,
    "offset": 0
  }
}
```

### 5. Request Manual Payout

```javascript
// Request manual payout (restricted: once per 30 days per currency)
POST /api/projects/507f1f77bcf86cd799439011/payouts/request-manual
Authorization: Bearer {user_token}
Content-Type: application/json

{
  "currency": "BTC"
}

// Response
{
  "success": true,
  "message": "Manual payout requested and processed",
  "data": {
    "payout": {
      "_id": "507f1f77bcf86cd799439031",
      "currency": "BTC",
      "netAmount": 0.0009,
      "txHash": "abc123def456...",
      "status": "completed"
    }
  }
}
```

### 6. Fee Dashboard

```javascript
// Get comprehensive fee dashboard
GET /api/projects/507f1f77bcf86cd799439011/fee-dashboard
Authorization: Bearer {user_token}

// Response
{
  "success": true,
  "data": {
    "feeSettings": {
      "BTC": {
        "enabled": true,
        "feeAmount": 0.00005,
        "payoutAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "addressVerified": true
      }
    },
    "balances": [
      {
        "currency": "BTC",
        "collectedAmount": 0.0005,
        "threshold": 0.001,
        "status": "accumulating",
        "totalPaidOut": 0.0009,
        "totalCollected": 0.0014
      }
    ],
    "recentPayouts": [
      {
        "_id": "507f1f77bcf86cd799439030",
        "currency": "BTC",
        "netAmount": 0.0009,
        "status": "completed",
        "createdAt": "2024-01-15T02:00:00Z"
      }
    ],
    "totalEarned": 0.0009
  }
}
```

### Important Notes

1. **Fee Validation**: Project fee must be less than platform minimum fee (cannot undercut platform)
2. **Address Verification**: Required before fees are collected (prevents fraud)
3. **Automated Payouts**: Bi-weekly schedule (Wed/Sat 2 AM UTC) when threshold is met
4. **Network Fees**: Deducted from payout amount (platform pays network fees)
5. **Manual Payouts**: Restricted to once per 30 days per currency, requires 25% of threshold
6. **Fee Collection**: Automatically collected during user withdrawals if enabled and verified

---

## Conclusion

This document provides a comprehensive overview of the MUDBASE Backend-as-a-Service platform, including:

- **Multiple Authentication Methods**: Local, OAuth, Magic Link, OTP, 2FA
- **Wallet as a Service**: Support for 7 cryptocurrencies with custom key support
- **Project Fee System**: Project owners can set their own fees with automated bi-weekly payouts
- **Billing System**: Nigerian payment gateways (Paystack, Flutterwave) with subscription management
- **Real-time Features**: Socket.IO-based chat system
- **Database Operations**: Flexible schema-based collections
- **Integration System**: 50+ API integrations
- **Security**: RBAC, encryption, rate limiting, audit logging

For more detailed API documentation, refer to the OpenAPI specification at `/api-docs`.

