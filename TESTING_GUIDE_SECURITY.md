# MUDBASE Security Testing Guide

> **Purpose**: Comprehensive security testing guide for identifying vulnerabilities  
> **Audience**: Security Engineers, Penetration Testers, QA Engineers  
> **Estimated Time**: 6-8 hours for complete security testing  
> **Prerequisites**: Complete [Functional Testing Guide](./TESTING_GUIDE_FUNCTIONAL.md) first

---

## Table of Contents

1. [Security Testing Overview](#security-testing-overview)
2. [Prerequisites](#prerequisites)
3. [IDOR (Insecure Direct Object Reference) Testing](#idor-insecure-direct-object-reference-testing)
4. [Improper Access Control Testing](#improper-access-control-testing)
5. [Privilege Escalation Testing](#privilege-escalation-testing)
6. [Race Condition Testing](#race-condition-testing)
7. [Input Validation Testing](#input-validation-testing)
8. [Authentication & Session Management Testing](#authentication--session-management-testing)
9. [Authorization Bypass Testing](#authorization-bypass-testing)
10. [Data Exposure Testing](#data-exposure-testing)
11. [API Rate Limiting Testing](#api-rate-limiting-testing)
12. [Security Checklist](#security-checklist)

---

## Security Testing Overview

This guide covers security testing for common vulnerabilities:

- **IDOR**: Unauthorized access to resources by manipulating IDs
- **Access Control**: Bypassing role/permission checks
- **Privilege Escalation**: Gaining higher privileges than authorized
- **Race Conditions**: Exploiting timing issues in concurrent requests
- **Input Validation**: Injection attacks, XSS, etc.
- **Authentication**: Session hijacking, token manipulation
- **Authorization**: Permission bypass attempts

---

## Prerequisites

### Required Tools

- **Burp Suite** or **OWASP ZAP**: For intercepting and modifying requests
- **Postman**: For API testing
- **cURL**: For command-line testing
- **JWT.io**: For decoding/encoding JWT tokens
- **MongoDB Compass**: For database verification
- **Multiple Test Accounts**: Different roles (customer, vendor, admin, etc.)

### Test Accounts Setup

Create test accounts for different roles:

```bash
# Account 1: Customer
POST {{baseUrl}}/api/auth/local/signup/customer
{
  "email": "customer-test@example.com",
  "password": "Customer123!",
  "firstName": "Customer",
  "lastName": "Test",
  "projectId": "{{projectId}}"
}
# Save: customerToken, customerUserId

# Account 2: Vendor
POST {{baseUrl}}/api/auth/local/signup/vendor
{
  "email": "vendor-test@example.com",
  "password": "Vendor123!",
  "firstName": "Vendor",
  "lastName": "Test",
  "projectId": "{{projectId}}"
}
# Save: vendorToken, vendorUserId

# Account 3: Admin (if possible)
# Save: adminToken, adminUserId
```

---

## IDOR (Insecure Direct Object Reference) Testing

### Test Case 1.1: Access Other User's Data by ID Manipulation

**Objective**: Verify users cannot access data belonging to other users by changing IDs in requests.

#### Step 1: Create Data as Customer A

```bash
POST {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data
Authorization: Bearer {{customerAToken}}
Content-Type: application/json

{
  "customerId": "{{customerAUserId}}",
  "items": [{"productId": "prod1", "quantity": 1}],
  "total": 29.99,
  "status": "pending"
}

# Response: 201 Created
# Save: orderId (e.g., "order123")
```

#### Step 2: Attempt to Access as Customer B

```bash
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data/order123
Authorization: Bearer {{customerBToken}}

# Expected: 403 Forbidden or 404 Not Found
# ❌ Vulnerability if: 200 OK with Customer A's order data
```

**Test Variations**:
- Try accessing orders, products, files, wallets belonging to other users
- Try accessing data from different organizations
- Try accessing data from different projects

**Expected Result**: All unauthorized access attempts should return 403 or 404.

---

### Test Case 1.2: Modify Other User's Data by ID Manipulation

**Objective**: Verify users cannot modify data belonging to other users.

#### Step 1: Attempt to Update Another User's Order

```bash
PATCH {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data/order123
Authorization: Bearer {{customerBToken}}
Content-Type: application/json

{
  "status": "cancelled"
}

# Expected: 403 Forbidden
# ❌ Vulnerability if: 200 OK (order updated)
```

#### Step 2: Attempt to Delete Another User's Data

```bash
DELETE {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data/order123
Authorization: Bearer {{customerBToken}}

# Expected: 403 Forbidden
# ❌ Vulnerability if: 200 OK (order deleted)
```

**Expected Result**: All unauthorized modification attempts should return 403.

---

### Test Case 1.3: Access Resources via Predictable IDs

**Objective**: Test if system uses predictable IDs that can be enumerated.

```bash
# Try accessing sequential IDs
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data/1
Authorization: Bearer {{customerAToken}}

GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data/2
Authorization: Bearer {{customerAToken}}

GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data/3
Authorization: Bearer {{customerAToken}}

# Expected: 404 for IDs not belonging to user
# ❌ Vulnerability if: Can access other users' data with sequential IDs
```

**Expected Result**: System should use non-sequential IDs (UUIDs, ObjectIds) and enforce access control.

---

## Improper Access Control Testing

### Test Case 2.1: Access Admin-Only Endpoints

**Objective**: Verify non-admin users cannot access admin-only endpoints.

```bash
# As Customer, try to access admin endpoints
GET {{baseUrl}}/api/orgs/{{orgId}}/members
Authorization: Bearer {{customerToken}}

# Expected: 403 Forbidden
# ❌ Vulnerability if: 200 OK with member list

GET {{baseUrl}}/api/orgs/{{orgId}}/role-elevation/pending
Authorization: Bearer {{customerToken}}

# Expected: 403 Forbidden
# ❌ Vulnerability if: 200 OK with pending requests
```

**Test Additional Admin Endpoints**:
- `DELETE /api/orgs/{orgId}/members/{userId}` - Remove members
- `POST /api/orgs/{orgId}/role-elevation/{requestId}/approve` - Approve role elevation
- `GET /api/projects/{projectId}/usage` - View usage statistics
- `POST /api/projects/{projectId}/backups` - Create backups

**Expected Result**: All admin-only endpoints should return 403 for non-admin users.

---

### Test Case 2.2: Access Collection Data Without Permissions

**Objective**: Verify users cannot access collections they don't have permissions for.

#### Step 1: Create Restricted Collection

```bash
POST {{baseUrl}}/api/projects/{{projectId}}/collections
Authorization: Bearer {{adminToken}}
Content-Type: application/json

{
  "name": "admin_only_data",
  "slug": "admin_only_data",
  "fields": [
    {"name": "secret", "type": "string", "required": true}
  ],
  "permissions": [
    {
      "role": "admin",
      "actions": ["create", "read", "update", "delete"]
    }
  ]
}
# Save: adminOnlyCollectionId
```

#### Step 2: Attempt Access as Customer

```bash
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{adminOnlyCollectionId}}/data
Authorization: Bearer {{customerToken}}

# Expected: 403 Forbidden
# ❌ Vulnerability if: 200 OK with data
```

**Expected Result**: Users without permissions should receive 403.

---

### Test Case 2.3: Bypass Collection-Level Permissions

**Objective**: Test if collection permissions can be bypassed.

```bash
# Create data in customer-only collection as vendor
POST {{baseUrl}}/api/projects/{{projectId}}/collections/{{customerOnlyCollectionId}}/data
Authorization: Bearer {{vendorToken}}
Content-Type: application/json

{
  "customerData": "sensitive information"
}

# Expected: 403 Forbidden
# ❌ Vulnerability if: 201 Created
```

**Test Variations**:
- Try creating data without required permissions
- Try updating data without update permission
- Try deleting data without delete permission
- Try reading data without read permission

**Expected Result**: All permission violations should return 403.

---

## Privilege Escalation Testing

### Test Case 3.1: Modify Role During Registration

**Objective**: Verify clients cannot assign themselves elevated roles during registration.

#### Test: Attempt to Set Admin Role in Request Body

```bash
POST {{baseUrl}}/api/auth/local/register
Content-Type: application/json

{
  "email": "hacker@example.com",
  "password": "HackerPass123!",
  "firstName": "Hacker",
  "lastName": "User",
  "projectId": "{{projectId}}",
  "role": "admin",
  "customRole": "admin"
}

# Expected: User created with default role (developer), not admin
# ❌ Vulnerability if: User created with admin role
```

**Verify in Response**:
```json
{
  "user": {
    "role": "developer",  // ✅ Should be default, not admin
    "customRole": null    // ✅ Should be null or default role
  }
}
```

**Expected Result**: Server should ignore client-supplied role fields and assign default roles.

---

### Test Case 3.2: Modify Role via Profile Update

**Objective**: Verify users cannot change their own role through profile updates.

```bash
# Attempt to update role in user profile collection
PATCH {{baseUrl}}/api/projects/{{projectId}}/collections/{{userProfilesCollectionId}}/data/{{profileId}}
Authorization: Bearer {{customerToken}}
Content-Type: application/json

{
  "role": "admin",
  "customRole": "admin",
  "userType": "admin"
}

# Expected: 403 Forbidden or field ignored
# ❌ Vulnerability if: Role updated to admin
```

**Verify**:
```bash
GET {{baseUrl}}/api/users/me
Authorization: Bearer {{customerToken}}

# Expected: Role still "developer", customRole still "customer"
# ❌ Vulnerability if: Role changed to admin
```

**Expected Result**: Role fields should be protected and cannot be modified by users.

---

### Test Case 3.3: Elevate Role via Role Elevation Endpoint (Unauthorized)

**Objective**: Verify users cannot approve their own role elevation requests.

```bash
# Create role elevation request
POST {{baseUrl}}/api/projects/{{projectId}}/role-elevation/request
Authorization: Bearer {{customerToken}}
Content-Type: application/json

{
  "roleSlug": "vendor"
}
# Save: requestId

# Attempt to approve own request (should require admin)
POST {{baseUrl}}/api/orgs/{{orgId}}/role-elevation/{{requestId}}/approve
Authorization: Bearer {{customerToken}}
Content-Type: application/json

{
  "approved": true
}

# Expected: 403 Forbidden
# ❌ Vulnerability if: 200 OK (request approved)
```

**Expected Result**: Only admins should be able to approve role elevation requests.

---

### Test Case 3.4: Assign Role to Others (Unauthorized)

**Objective**: Verify non-admin users cannot assign roles to other users.

```bash
POST {{baseUrl}}/api/orgs/{{orgId}}/users/{{otherUserId}}/role
Authorization: Bearer {{customerToken}}
Content-Type: application/json

{
  "roleSlug": "vendor"
}

# Expected: 403 Forbidden
# ❌ Vulnerability if: 200 OK (role assigned)
```

**Expected Result**: Only admins/owners should be able to assign roles.

---

### Test Case 3.5: Bypass Multi-Role Feature Restrictions

**Objective**: Verify role-based signup respects role configuration.

```bash
# Disable vendor role
PATCH {{baseUrl}}/api/projects/{{projectId}}/multi-role/roles/vendor/toggle
Authorization: Bearer {{adminToken}}
Content-Type: application/json

{
  "isEnabled": false
}

# Attempt to signup with disabled role
POST {{baseUrl}}/api/auth/local/signup/vendor
Content-Type: application/json

{
  "email": "vendor2@example.com",
  "password": "Vendor123!",
  "firstName": "Vendor2",
  "lastName": "User",
  "projectId": "{{projectId}}"
}

# Expected: 404 Not Found (role not available)
# ❌ Vulnerability if: 201 Created (user created with vendor role)
```

**Expected Result**: Disabled roles should not be available for signup.

---

## Race Condition Testing

### Test Case 4.1: Concurrent Role Assignment

**Objective**: Test for race conditions when multiple admins assign roles simultaneously.

#### Setup: Create Script for Concurrent Requests

```bash
# Script 1: Admin A assigns role
POST {{baseUrl}}/api/orgs/{{orgId}}/users/{{targetUserId}}/role
Authorization: Bearer {{adminAToken}}
Content-Type: application/json

{
  "roleSlug": "vendor"
}

# Script 2: Admin B assigns different role (run simultaneously)
POST {{baseUrl}}/api/orgs/{{orgId}}/users/{{targetUserId}}/role
Authorization: Bearer {{adminBToken}}
Content-Type: application/json

{
  "roleSlug": "rider"
}

# Run both requests simultaneously (within 100ms)
# Expected: One succeeds, one may fail or both handled correctly
# ❌ Vulnerability if: Both succeed causing inconsistent state
```

**Verify Final State**:
```bash
GET {{baseUrl}}/api/orgs/{{orgId}}/users/{{targetUserId}}/permissions
Authorization: Bearer {{adminToken}}

# Expected: User has one consistent role (either vendor or rider)
# ❌ Vulnerability if: User has both roles or inconsistent state
```

**Expected Result**: System should handle concurrent updates correctly (atomic transactions).

---

### Test Case 4.2: Concurrent Order Creation (Double Spending)

**Objective**: Test for race conditions in order creation/payment processing.

```bash
# Create two identical orders simultaneously
POST {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data
Authorization: Bearer {{customerToken}}
Content-Type: application/json

{
  "customerId": "{{customerUserId}}",
  "items": [{"productId": "prod1", "quantity": 1}],
  "total": 29.99,
  "paymentIntentId": "pi_same_payment_intent_id"
}

# Send same request again immediately (within 100ms)
# Expected: Second request should be rejected or handled correctly
# ❌ Vulnerability if: Both orders created with same payment (double spending)
```

**Verify**:
```bash
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data?filter={"paymentIntentId":"pi_same_payment_intent_id"}
Authorization: Bearer {{customerToken}}

# Expected: Only one order with this paymentIntentId
# ❌ Vulnerability if: Multiple orders with same paymentIntentId
```

**Expected Result**: System should prevent duplicate processing of same payment.

---

### Test Case 4.3: Concurrent File Uploads

**Objective**: Test for race conditions in file uploads.

```bash
# Upload same file twice simultaneously
POST {{baseUrl}}/api/files/{{projectId}}/upload
Authorization: Bearer {{customerToken}}
Content-Type: multipart/form-data

Form Data:
- file: [same file]
- bucket: test-bucket

# Send same request again immediately
# Expected: Both uploads handled correctly (separate file IDs)
# ❌ Vulnerability if: One overwrites the other or causes errors
```

**Expected Result**: System should handle concurrent uploads correctly.

---

### Test Case 4.4: Concurrent Wallet Transactions

**Objective**: Test for race conditions in wallet transactions.

```bash
# Create two withdrawal requests simultaneously
POST {{baseUrl}}/api/wallet/projects/{{projectId}}/wallets/{{walletId}}/withdraw
Authorization: Bearer {{customerToken}}
Content-Type: application/json

{
  "currency": "BTC",
  "amount": "0.001",
  "toAddress": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh"
}

# Send same request again immediately
# Expected: Second request should check balance and fail if insufficient
# ❌ Vulnerability if: Both transactions process causing negative balance
```

**Verify Balance**:
```bash
GET {{baseUrl}}/api/wallet/projects/{{projectId}}/wallets/{{walletId}}/balance
Authorization: Bearer {{customerToken}}

# Expected: Balance is correct (not negative)
# ❌ Vulnerability if: Balance is negative or incorrect
```

**Expected Result**: System should use atomic transactions to prevent double spending.

---

## Input Validation Testing

### Test Case 5.1: SQL Injection (NoSQL)

**Objective**: Test for NoSQL injection vulnerabilities.

```bash
# Attempt NoSQL injection in filter
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{usersCollectionId}}/data?filter={"email":{"$ne":null},"$where":"this.password"}
Authorization: Bearer {{token}}

# Expected: 400 Bad Request (invalid filter)
# ❌ Vulnerability if: Returns user data including passwords

# Attempt MongoDB operator injection
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{usersCollectionId}}/data?filter={"$where":"function(){return true}"}
Authorization: Bearer {{token}}

# Expected: 400 Bad Request
# ❌ Vulnerability if: Executes and returns all data
```

**Expected Result**: System should sanitize and validate all input, rejecting dangerous operators.

---

### Test Case 5.2: XSS (Cross-Site Scripting)

**Objective**: Test for XSS vulnerabilities in stored data.

```bash
# Attempt to store XSS payload
POST {{baseUrl}}/api/projects/{{projectId}}/collections/{{usersCollectionId}}/data
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "<script>alert('XSS')</script>",
  "email": "test@example.com",
  "bio": "<img src=x onerror=alert('XSS')>"
}

# Expected: Data stored, but rendered safely (HTML escaped)
# ❌ Vulnerability if: Script executes when data is displayed
```

**Expected Result**: System should escape HTML/JavaScript in stored data when rendering.

---

### Test Case 5.3: Command Injection

**Objective**: Test for command injection vulnerabilities.

```bash
# Attempt command injection in file operations
POST {{baseUrl}}/api/files/{{projectId}}/upload
Authorization: Bearer {{token}}
Content-Type: multipart/form-data

Form Data:
- file: [test file]
- bucket: "test; rm -rf /"
- fileName: "../../etc/passwd"

# Expected: Invalid input rejected
# ❌ Vulnerability if: Commands are executed
```

**Expected Result**: System should sanitize file names and paths.

---

### Test Case 5.4: Path Traversal

**Objective**: Test for path traversal vulnerabilities.

```bash
# Attempt path traversal in file access
GET {{baseUrl}}/api/files/../../../etc/passwd/download
Authorization: Bearer {{token}}

# Expected: 404 Not Found or 403 Forbidden
# ❌ Vulnerability if: Server files are accessed

# Attempt path traversal in collection data
GET {{baseUrl}}/api/projects/{{projectId}}/collections/../../admin/data
Authorization: Bearer {{token}}

# Expected: 404 Not Found
# ❌ Vulnerability if: Accesses unauthorized collections
```

**Expected Result**: System should prevent path traversal attacks.

---

## CAPTCHA Bypass Testing

### Test Case 5.1: CAPTCHA Token Reuse (Replay Attack)

**Objective**: Verify CAPTCHA tokens cannot be reused multiple times.

```bash
# Step 1: Get valid CAPTCHA token from frontend
# Step 2: Use token for registration
POST {{baseUrl}}/api/auth/local/register
{
  "email": "test1@example.com",
  "password": "TestPassword123!",
  "projectId": "{{projectId}}",
  "captcha": "03AGdBq24PjFyF8Z..." // Valid token
}

# Expected: 201 Created

# Step 3: Attempt to reuse same token
POST {{baseUrl}}/api/auth/local/register
{
  "email": "test2@example.com",
  "password": "TestPassword123!",
  "projectId": "{{projectId}}",
  "captcha": "03AGdBq24PjFyF8Z..." // Same token
}

# Expected: 400 Bad Request (CAPTCHA verification failed)
# ❌ Vulnerability if: 201 Created (token reused successfully)
```

**Expected Result**: CAPTCHA tokens should be single-use and verified with Google's API each time.

---

### Test Case 5.2: CAPTCHA Bypass (Missing Token)

**Objective**: Verify authentication fails when CAPTCHA is enabled but token is missing.

```bash
# With CAPTCHA enabled for project
POST {{baseUrl}}/api/auth/local/register
{
  "email": "bypass-test@example.com",
  "password": "TestPassword123!",
  "projectId": "{{projectId}}"
  // No captcha field
}

# Expected: 400 Bad Request
# {
#   "error": "CAPTCHA verification required",
#   "captchaRequired": true,
#   "siteKey": "..."
# }
# ❌ Vulnerability if: 201 Created (registration succeeds without CAPTCHA)
```

**Expected Result**: When CAPTCHA is enabled, requests without CAPTCHA tokens should be rejected.

---

### Test Case 5.3: Invalid CAPTCHA Token

**Objective**: Verify invalid/fake CAPTCHA tokens are rejected.

```bash
POST {{baseUrl}}/api/auth/local/register
{
  "email": "invalid-captcha@example.com",
  "password": "TestPassword123!",
  "projectId": "{{projectId}}",
  "captcha": "invalid_token_12345"
}

# Expected: 400 Bad Request
# {
#   "error": "CAPTCHA verification failed",
#   "details": ["invalid-input-response"]
# }
# ❌ Vulnerability if: 201 Created (invalid token accepted)
```

**Expected Result**: Invalid CAPTCHA tokens should be rejected by Google's API.

---

### Test Case 5.4: CAPTCHA Score Manipulation (reCAPTCHA v3)

**Objective**: Verify low-score CAPTCHA tokens are rejected when minScore threshold is set.

```bash
# Project configured with minScore: 0.7
# Attempt registration with low-score token (score < 0.7)
POST {{baseUrl}}/api/auth/local/register
{
  "email": "low-score@example.com",
  "password": "TestPassword123!",
  "projectId": "{{projectId}}",
  "captcha": "03AGdBq24PjFyF8Z..." // Token with score 0.3
}

# Expected: 400 Bad Request
# {
#   "error": "CAPTCHA score too low",
#   "score": 0.3,
#   "minScore": 0.7
# }
# ❌ Vulnerability if: 201 Created (low score accepted)
```

**Expected Result**: CAPTCHA tokens with scores below the minimum threshold should be rejected.

---

### Test Case 5.5: CAPTCHA Disabled Bypass

**Objective**: Verify that when CAPTCHA is disabled, it can be bypassed (this is expected behavior).

```bash
# CAPTCHA disabled for project
POST {{baseUrl}}/api/auth/local/register
{
  "email": "no-captcha@example.com",
  "password": "TestPassword123!",
  "projectId": "{{projectId}}"
  // No captcha field - should work
}

# Expected: 201 Created (CAPTCHA disabled, bypass is expected)
```

**Expected Result**: When CAPTCHA is disabled, requests should work without CAPTCHA tokens.

---

## Authentication & Session Management Testing

### Test Case 6.1: Token Manipulation

**Objective**: Verify tokens cannot be tampered with.

#### Step 1: Decode JWT Token

```bash
# Use jwt.io to decode token
# Token structure: header.payload.signature
```

#### Step 2: Modify Token and Attempt Usage

```bash
# Modify payload (e.g., change userId to another user's ID)
# Keep same signature
# Expected: 401 Unauthorized (invalid signature)
# ❌ Vulnerability if: Request succeeds with modified token
```

**Expected Result**: Token signatures should be validated; modified tokens should be rejected.

---

### Test Case 6.2: Token Replay

**Objective**: Test if expired or revoked tokens can be reused.

```bash
# Use expired token
GET {{baseUrl}}/api/users/me
Authorization: Bearer {{expiredToken}}

# Expected: 401 Unauthorized
# ❌ Vulnerability if: Request succeeds

# Use revoked token (logout)
POST {{baseUrl}}/api/auth/logout
Authorization: Bearer {{token}}

# Try to use same token
GET {{baseUrl}}/api/users/me
Authorization: Bearer {{revokedToken}}

# Expected: 401 Unauthorized
# ❌ Vulnerability if: Request succeeds
```

**Expected Result**: Expired and revoked tokens should be rejected.

---

### Test Case 6.3: Session Fixation

**Objective**: Test for session fixation vulnerabilities.

```bash
# Login and get token
POST {{baseUrl}}/api/auth/local/login
{
  "email": "test@example.com",
  "password": "Password123!",
  "projectId": "{{projectId}}"
}
# Save: token1

# Change password
POST {{baseUrl}}/api/auth/password-reset/{{resetToken}}
{
  "password": "NewPassword123!"
}

# Try to use old token
GET {{baseUrl}}/api/users/me
Authorization: Bearer {{token1}}

# Expected: 401 Unauthorized (old session invalidated)
# ❌ Vulnerability if: Request succeeds
```

**Expected Result**: Password changes should invalidate existing sessions.

---

## Authorization Bypass Testing

### Test Case 7.1: Bypass Collection Permissions via Direct API

**Objective**: Test if collection permissions can be bypassed.

```bash
# Create collection with customer-only read permission
# Try to update as customer
PATCH {{baseUrl}}/api/projects/{{projectId}}/collections/{{readOnlyCollectionId}}/data/{{documentId}}
Authorization: Bearer {{customerToken}}
Content-Type: application/json

{
  "field": "modified value"
}

# Expected: 403 Forbidden
# ❌ Vulnerability if: 200 OK (update succeeded)
```

**Expected Result**: Collection permissions should be enforced at API level.

---

### Test Case 7.2: Access Admin Panel as Non-Admin

**Objective**: Test if admin-only UI endpoints can be accessed.

```bash
# Attempt to access admin dashboard endpoints
GET {{baseUrl}}/api/admin/dashboard
Authorization: Bearer {{customerToken}}

# Expected: 403 Forbidden
# ❌ Vulnerability if: 200 OK with admin data

GET {{baseUrl}}/api/admin/users
Authorization: Bearer {{customerToken}}

# Expected: 403 Forbidden
# ❌ Vulnerability if: 200 OK with user list
```

**Expected Result**: Admin endpoints should require admin role.

---

### Test Case 7.3: Bypass Multi-Organization Isolation

**Objective**: Verify users cannot access data from other organizations.

```bash
# User from Org A tries to access Org B's data
GET {{baseUrl}}/api/projects/{{orgBProjectId}}/collections/{{collectionId}}/data
Authorization: Bearer {{orgAToken}}

# Expected: 403 Forbidden or 404 Not Found
# ❌ Vulnerability if: 200 OK with Org B's data
```

**Expected Result**: Row-Level Security (RLS) should prevent cross-organization access.

---

## Data Exposure Testing

### Test Case 8.1: Sensitive Data in Responses

**Objective**: Verify sensitive data is not exposed in API responses.

```bash
# Register user and check response
POST {{baseUrl}}/api/auth/local/register
{
  "email": "test@example.com",
  "password": "Password123!",
  "projectId": "{{projectId}}"
}

# Verify response doesn't contain:
# ❌ Password (even hashed)
# ❌ Internal IDs
# ❌ Secrets/API keys
# ✅ Only safe user information

# Get user profile
GET {{baseUrl}}/api/users/me
Authorization: Bearer {{token}}

# Verify response doesn't expose:
# ❌ Password hashes
# ❌ Internal tokens
# ❌ Encryption keys
```

**Expected Result**: Only safe, necessary data should be returned.

---

### Test Case 8.2: Information Disclosure via Error Messages

**Objective**: Test if error messages leak sensitive information.

```bash
# Trigger various errors and check responses
POST {{baseUrl}}/api/auth/local/login
{
  "email": "nonexistent@example.com",
  "password": "wrong",
  "projectId": "{{projectId}}"
}

# Expected: Generic error message
# ❌ Vulnerability if: Error reveals user existence, database structure, stack traces

# Invalid collection ID
GET {{baseUrl}}/api/projects/{{projectId}}/collections/invalid_collection/data
Authorization: Bearer {{token}}

# Expected: Generic 404 error
# ❌ Vulnerability if: Error reveals internal structure or paths
```

**Expected Result**: Error messages should be generic and not reveal sensitive information.

---

### Test Case 8.3: Data Leakage in Lists

**Objective**: Verify list endpoints don't leak unauthorized data.

```bash
# Get all users (as customer)
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{usersCollectionId}}/data
Authorization: Bearer {{customerToken}}

# Expected: Only user's own data or empty list (if no read permission)
# ❌ Vulnerability if: Returns all users' data

# Get all orders (as vendor)
GET {{baseUrl}}/api/projects/{{projectId}}/collections/{{ordersCollectionId}}/data
Authorization: Bearer {{vendorToken}}

# Expected: Only vendor's orders
# ❌ Vulnerability if: Returns all orders from all vendors
```

**Expected Result**: List endpoints should respect permissions and filters.

---

## API Rate Limiting Testing

### Test Case 9.1: Authentication Rate Limiting

**Objective**: Test if rate limiting prevents brute force attacks.

```bash
# Attempt multiple failed logins
for i in {1..20}; do
  POST {{baseUrl}}/api/auth/local/login
  {
    "email": "test@example.com",
    "password": "wrongpassword",
    "projectId": "{{projectId}}"
  }
done

# Expected: After 5-10 attempts, should return 429 Too Many Requests
# ❌ Vulnerability if: No rate limiting (allows unlimited attempts)
```

**Expected Result**: Rate limiting should prevent brute force attacks.

---

### Test Case 9.2: API Endpoint Rate Limiting

**Objective**: Test if API endpoints have rate limiting.

```bash
# Send many requests to an endpoint
for i in {1..100}; do
  GET {{baseUrl}}/api/users/me
  Authorization: Bearer {{token}}
done

# Expected: After threshold, should return 429 Too Many Requests
# ❌ Vulnerability if: No rate limiting (allows unlimited requests)
```

**Expected Result**: API endpoints should have appropriate rate limits.

---

### Test Case 9.3: Bypass Rate Limiting

**Objective**: Test if rate limiting can be bypassed.

```bash
# Try different techniques:
# 1. Change IP address (if applicable)
# 2. Rotate tokens
# 3. Use different endpoints

# Expected: Rate limiting should be enforced per user/IP
# ❌ Vulnerability if: Easy to bypass
```

**Expected Result**: Rate limiting should be properly implemented and not easily bypassed.

---

## Security Checklist

Use this checklist to ensure all security tests are performed:

### IDOR Testing
- [ ] Access other users' data by ID manipulation
- [ ] Modify other users' data by ID manipulation
- [ ] Delete other users' data by ID manipulation
- [ ] Enumerate resources via predictable IDs
- [ ] Access data from other organizations
- [ ] Access data from other projects

### Access Control Testing
- [ ] Access admin-only endpoints as non-admin
- [ ] Access collection data without permissions
- [ ] Bypass collection-level permissions
- [ ] Access protected file resources
- [ ] Access wallet data of other users

### Privilege Escalation Testing
- [ ] Modify role during registration
- [ ] Modify role via profile update
- [ ] Elevate role via role elevation endpoint (unauthorized)
- [ ] Assign role to others (unauthorized)
- [ ] Bypass multi-role feature restrictions
- [ ] Access admin panel as non-admin

### Race Condition Testing
- [ ] Concurrent role assignments
- [ ] Concurrent order creation (double spending)
- [ ] Concurrent file uploads
- [ ] Concurrent wallet transactions
- [ ] Concurrent payment processing

### Input Validation Testing
- [ ] SQL/NoSQL injection
- [ ] XSS (Cross-Site Scripting)
- [ ] Command injection
- [ ] Path traversal
- [ ] Buffer overflow (if applicable)
- [ ] XML/XXE injection (if applicable)

### Authentication Testing
- [ ] Token manipulation
- [ ] Token replay (expired/revoked)
- [ ] Session fixation
- [ ] Session hijacking
- [ ] Weak password policy
- [ ] Password reset token security
- [ ] CAPTCHA bypass attempts
- [ ] CAPTCHA token reuse (replay attacks)
- [ ] CAPTCHA score manipulation (v3)

### Authorization Testing
- [ ] Bypass collection permissions
- [ ] Access admin panel as non-admin
- [ ] Bypass multi-organization isolation
- [ ] Access resources without proper role
- [ ] Modify permissions directly

### Data Exposure Testing
- [ ] Sensitive data in responses
- [ ] Information disclosure via errors
- [ ] Data leakage in lists
- [ ] Credentials in logs
- [ ] API keys in responses

### Rate Limiting Testing
- [ ] Authentication rate limiting
- [ ] API endpoint rate limiting
- [ ] Bypass rate limiting attempts

---

## Reporting Security Issues

When reporting security vulnerabilities, include:

1. **Severity**: Critical / High / Medium / Low
2. **Vulnerability Type**: IDOR, Privilege Escalation, etc.
3. **Description**: Clear explanation of the vulnerability
4. **Steps to Reproduce**: Detailed steps with requests/responses
5. **Expected Behavior**: What should happen
6. **Actual Behavior**: What actually happens
7. **Impact**: Potential consequences
8. **Proof of Concept**: If applicable
9. **Suggested Fix**: If you have recommendations

**Important**: Do not publicly disclose vulnerabilities. Report them privately to the security team.

---

## Tools and Resources

### Recommended Tools

- **Burp Suite**: Professional web security testing
- **OWASP ZAP**: Free security testing tool
- **Postman**: API testing and automation
- **JWT.io**: JWT token analysis
- **MongoDB Compass**: Database inspection
- **cURL**: Command-line HTTP client

### Security Testing Resources

- **OWASP Top 10**: https://owasp.org/www-project-top-ten/
- **OWASP API Security Top 10**: https://owasp.org/www-project-api-security/
- **OWASP Testing Guide**: https://owasp.org/www-project-web-security-testing-guide/

---

**Last Updated**: December 2024  
**Version**: 1.0.0  
**Classification**: Confidential - For Internal Testing Only

