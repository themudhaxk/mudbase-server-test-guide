# MUDBASE New Features & Workflows (December 2024)

> **📅 Released**: December 2024  
> **📚 Supplement to**: `WORKFLOW_SIMULATION.md`  
> **✨ New Features**: 24+ major enhancements

This document covers all new features added to MUDBASE in December 2024. For core features, see [WORKFLOW_SIMULATION.md](./WORKFLOW_SIMULATION.md).

---

## Table of Contents

1. [Multi-Role & Custom RBAC (NEW)](#multi-role--custom-rbac-new) ⭐
2. [Multi-Role Feature - Ready-to-Use System (NEW)](#multi-role-feature---ready-to-use-system-new) ⭐⭐
3. [Anonymous Authentication](#anonymous-authentication)
3. [OAuth Account Linking](#oauth-account-linking)
4. [GDPR Compliance Tools](#gdpr-compliance-tools)
5. [Stripe Invoicing](#stripe-invoicing)
6. [Row-Level Security (RLS)](#row-level-security-rls)
7. [Backup & Restore](#backup--restore)
8. [Virus Scanning](#virus-scanning)
9. [Per-File RBAC](#per-file-rbac)
10. [Real-time Analytics](#real-time-analytics)
11. [Webhook Transformations](#webhook-transformations)
12. [Compliance Automation](#compliance-automation)
13. [OpenTelemetry Tracing](#opentelemetry-tracing)

---

## Multi-Role & Custom RBAC (NEW)

**Endpoint**: `/api/orgs/:orgId/roles`  
**Use Case**: Multi-tenant SaaS, marketplaces, delivery apps, e-commerce platforms with multiple user types

### Overview

MUDBASE provides a comprehensive **Role-Based Access Control (RBAC)** system that supports:
- ✅ **System Roles**: owner, admin, developer, viewer (built-in)
- ✅ **Custom Roles**: Create unlimited application-specific roles
- ✅ **Granular Permissions**: Resource-level permissions (project, collection, data, file, etc.)
- ✅ **Role Hierarchy**: Numeric hierarchy for role precedence
- ✅ **Multi-Role Users**: Users can have both system + custom roles
- ✅ **Permission Inheritance**: Custom roles add to system role permissions

### Common Use Cases

**1. E-commerce Platform:**
- `customer` - Browse products, place orders
- `seller` - Manage own products, view own sales
- `support` - View orders, help customers
- `admin` - Manage all users and products

**2. Delivery App (Uber/DoorDash-style):**
- `customer` - Place orders, track delivery
- `rider` - Accept orders, update delivery status
- `restaurant_partner` - Manage menu, view orders
- `support_agent` - Assist users, resolve issues
- `super_admin` - Full platform control

**3. Multi-Tenant SaaS:**
- `org_owner` - Manage organization settings
- `project_admin` - Manage specific project
- `team_member` - Collaborate on project
- `guest` - Read-only access

### System Roles (Built-in)

These roles control **platform-level** access:

```javascript
// System roles with hierarchy
owner (100)      → Full access to organization and all projects
admin (80)       → Administrative access to projects and data
developer (60)   → Development access (CRUD operations)
viewer (20)      → Read-only access
```

**System Role Permissions:**

| Resource   | owner | admin | developer | viewer |
|------------|-------|-------|-----------|--------|
| org        | ✅ All | ✅ Read/Update | ✅ Read | ✅ Read |
| project    | ✅ All | ✅ All | ✅ CRUD | ✅ Read |
| collection | ✅ All | ✅ All | ✅ All | ✅ Read |
| data       | ✅ All | ✅ All | ✅ All | ✅ Read |
| file       | ✅ All | ✅ All | ✅ All | ✅ Read |
| api_key    | ✅ All | ✅ CRUD | ✅ Create/Read/Update | ✅ Read |
| member     | ✅ All | ✅ Invite/Remove/Update | ❌ | ❌ |
| role       | ✅ All | ✅ Read | ✅ Read | ✅ Read |
| billing    | ✅ All | ✅ Read/Update | ❌ | ❌ |

---

### Custom Roles (Application-Specific)

Custom roles define **application-level** permissions for your specific use case.

### 1. Create Custom Role

```javascript
POST /api/orgs/507f1f77bcf86cd799439013/roles
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "Support Agent",
  "description": "Customer support team member",
  "hierarchy": 40,
  "permissions": [
    {
      "resource": "data",
      "actions": ["read", "update"],
      "conditions": {
        "collection": ["orders", "customers"],
        "statusField": "status"
      }
    },
    {
      "resource": "file",
      "actions": ["read"]
    }
  ]
}

// Response
{
  "message": "Role created successfully",
  "role": {
    "_id": "role123",
    "name": "Support Agent",
    "slug": "support_agent",
    "description": "Customer support team member",
    "org": "507f1f77bcf86cd799439013",
    "permissions": [...],
    "hierarchy": 40,
    "isSystem": false,
    "isActive": true,
    "createdBy": "user123",
    "createdAt": "2024-12-16T10:00:00Z"
  }
}
```

### 2. List All Roles

```javascript
GET /api/orgs/507f1f77bcf86cd799439013/roles
Authorization: Bearer {token}

// Response
{
  "roles": [
    {
      "_id": "role123",
      "name": "Support Agent",
      "slug": "support_agent",
      "hierarchy": 40,
      "permissions": [...],
      "isSystem": false
    },
    {
      "_id": "role456",
      "name": "Rider",
      "slug": "rider",
      "hierarchy": 30,
      "permissions": [...],
      "isSystem": false
    },
    {
      "_id": "role789",
      "name": "Restaurant Partner",
      "slug": "restaurant_partner",
      "hierarchy": 35,
      "permissions": [...],
      "isSystem": false
    }
  ],
  "total": 3
}
```

### 3. Assign Custom Role to User

```javascript
// User has TWO roles:
// - System role: "developer" (platform access)
// - Custom role: "support_agent" (application access)

POST /api/orgs/507f1f77bcf86cd799439013/users/user456/role
Authorization: Bearer {token}
Content-Type: application/json

{
  "roleSlug": "support_agent"
}

// Response
{
  "message": "Role assigned successfully",
  "user": {
    "_id": "user456",
    "email": "agent@company.com",
    "firstName": "Sarah",
    "lastName": "Johnson",
    "role": "developer",          // System role
    "customRole": "support_agent"  // Custom role
  }
}
```

### 4. Check User Permissions

```javascript
GET /api/orgs/507f1f77bcf86cd799439013/users/user456/permissions
Authorization: Bearer {token}

// Response
{
  "user": {
    "_id": "user456",
    "email": "agent@company.com",
    "role": "developer",
    "customRole": "support_agent"
  },
  "permissions": {
    "system": [
      "project:read",
      "project:create",
      "project:update",
      "collection:*",
      "data:*",
      "file:*",
      "api_key:create",
      "api_key:read",
      "api_key:update"
    ],
    "custom": [
      "data:read",
      "data:update",
      "file:read"
    ],
    "combined": [
      "project:read",
      "project:create",
      "project:update",
      "collection:*",
      "data:*",
      "file:*",
      "api_key:create",
      "api_key:read",
      "api_key:update"
    ]
  }
}
```

### 5. Update Role Permissions

```javascript
PUT /api/orgs/507f1f77bcf86cd799439013/roles/role123
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "Senior Support Agent",
  "permissions": [
    {
      "resource": "data",
      "actions": ["read", "update", "delete"],
      "conditions": {
        "collection": ["orders", "customers", "refunds"]
      }
    },
    {
      "resource": "file",
      "actions": ["read", "create"]
    }
  ],
  "hierarchy": 45
}

// Response
{
  "message": "Role updated successfully",
  "role": {
    "_id": "role123",
    "name": "Senior Support Agent",
    "slug": "support_agent",
    // ... updated fields
  }
}
```

### 6. Get Users with Specific Role

```javascript
GET /api/orgs/507f1f77bcf86cd799439013/roles/support_agent/users
Authorization: Bearer {token}

// Response
{
  "role": {
    "name": "Support Agent",
    "slug": "support_agent",
    "description": "Customer support team member"
  },
  "users": [
    {
      "_id": "user456",
      "email": "agent1@company.com",
      "firstName": "Sarah",
      "lastName": "Johnson",
      "role": "developer",
      "customRole": "support_agent",
      "lastLogin": "2024-12-16T09:00:00Z"
    },
    {
      "_id": "user789",
      "email": "agent2@company.com",
      "firstName": "Mike",
      "lastName": "Chen",
      "role": "developer",
      "customRole": "support_agent",
      "lastLogin": "2024-12-16T08:30:00Z"
    }
  ],
  "total": 2
}
```

### 7. Remove Custom Role

```javascript
DELETE /api/orgs/507f1f77bcf86cd799439013/users/user456/role
Authorization: Bearer {token}

// Response
{
  "message": "Custom role removed successfully",
  "user": {
    "_id": "user456",
    "email": "agent@company.com",
    "role": "developer",      // System role remains
    "customRole": null        // Custom role removed
  },
  "previousRole": "support_agent"
}
```

### 8. Delete Custom Role

```javascript
DELETE /api/orgs/507f1f77bcf86cd799439013/roles/role123
Authorization: Bearer {token}

// Response (if successful)
{
  "message": "Role deleted successfully",
  "deletedRole": {
    "_id": "role123",
    "name": "Support Agent",
    "slug": "support_agent"
  }
}

// Response (if users assigned)
{
  "error": "Cannot delete role",
  "message": "5 user(s) currently have this role. Reassign users before deleting.",
  "usersCount": 5
}
```

---

## Real-World Example: Delivery App

Let's build a full RBAC setup for a delivery app like Uber Eats or DoorDash.

### Step 1: Define Roles

```javascript
// 1. Create "Customer" role
POST /api/orgs/{orgId}/roles
{
  "name": "Customer",
  "description": "End user who places orders",
  "hierarchy": 10,
  "permissions": [
    {
      "resource": "data",
      "actions": ["read", "create"],
      "conditions": { "collection": ["restaurants", "menu_items"] }
    },
    {
      "resource": "data",
      "actions": ["read", "create", "update"],
      "conditions": { "collection": ["orders"], "owner": "$userId" }
    }
  ]
}

// 2. Create "Rider" role
POST /api/orgs/{orgId}/roles
{
  "name": "Rider",
  "description": "Delivery driver",
  "hierarchy": 30,
  "permissions": [
    {
      "resource": "data",
      "actions": ["read", "update"],
      "conditions": { 
        "collection": ["orders"], 
        "status": ["assigned", "picked_up", "in_transit"]
      }
    },
    {
      "resource": "file",
      "actions": ["create"],
      "conditions": { "type": ["delivery_proof"] }
    }
  ]
}

// 3. Create "Restaurant Partner" role
POST /api/orgs/{orgId}/roles
{
  "name": "Restaurant Partner",
  "description": "Restaurant owner/manager",
  "hierarchy": 40,
  "permissions": [
    {
      "resource": "data",
      "actions": ["read", "update"],
      "conditions": { 
        "collection": ["orders"], 
        "restaurantId": "$userRestaurantId"
      }
    },
    {
      "resource": "data",
      "actions": ["create", "read", "update", "delete"],
      "conditions": { 
        "collection": ["menu_items", "restaurant_profile"],
        "restaurantId": "$userRestaurantId"
      }
    }
  ]
}

// 4. Create "Support Agent" role
POST /api/orgs/{orgId}/roles
{
  "name": "Support Agent",
  "description": "Customer support team",
  "hierarchy": 50,
  "permissions": [
    {
      "resource": "data",
      "actions": ["read", "update"],
      "conditions": { 
        "collection": ["orders", "customers", "riders", "restaurants"]
      }
    },
    {
      "resource": "data",
      "actions": ["create"],
      "conditions": { "collection": ["refunds", "support_tickets"] }
    }
  ]
}

// 5. Create "Super Admin" role
POST /api/orgs/{orgId}/roles
{
  "name": "Super Admin",
  "description": "Platform administrator",
  "hierarchy": 90,
  "permissions": [
    {
      "resource": "data",
      "actions": ["create", "read", "update", "delete"]
    },
    {
      "resource": "file",
      "actions": ["create", "read", "update", "delete"]
    },
    {
      "resource": "member",
      "actions": ["invite", "remove", "update_role"]
    }
  ]
}
```

### Step 2: Assign Roles to Users

```javascript
// Assign "customer" role to end user
POST /api/orgs/{orgId}/users/user001/role
{
  "roleSlug": "customer"
}

// Assign "rider" role to delivery driver
POST /api/orgs/{orgId}/users/user002/role
{
  "roleSlug": "rider"
}

// Assign "restaurant_partner" role to restaurant owner
POST /api/orgs/{orgId}/users/user003/role
{
  "roleSlug": "restaurant_partner"
}

// Assign "support_agent" role to support team
POST /api/orgs/{orgId}/users/user004/role
{
  "roleSlug": "support_agent"
}

// Assign "super_admin" role to platform admin
POST /api/orgs/{orgId}/users/user005/role
{
  "roleSlug": "super_admin"
}
```

### Step 3: Permission Checking in Your App

When a user makes a request, their permissions are automatically checked:

```javascript
// Customer trying to view orders
GET /api/projects/{projectId}/data/orders?userId=user001
// ✅ Allowed - can see their own orders only

// Rider trying to update order status
PATCH /api/projects/{projectId}/data/orders/order123
{
  "status": "delivered",
  "deliveryProof": "photo_url"
}
// ✅ Allowed - can update assigned orders

// Restaurant partner trying to update menu
PATCH /api/projects/{projectId}/data/menu_items/item456
{
  "price": 12.99,
  "available": true
}
// ✅ Allowed - can manage their own menu items

// Support agent viewing all orders
GET /api/projects/{projectId}/data/orders
// ✅ Allowed - can view all orders for support

// Customer trying to delete a restaurant
DELETE /api/projects/{projectId}/data/restaurants/rest789
// ❌ Denied - no delete permission on restaurants
```

---

## Permission System Details

### Resource Types

- `org` - Organization settings
- `project` - Projects and configurations
- `collection` - Database collections/schemas
- `data` - Actual data records
- `file` - File storage
- `api_key` - API key management
- `member` - User management
- `role` - Role management
- `billing` - Billing and subscriptions

### Action Types

- `create` - Create new records
- `read` - View/query records
- `update` - Modify existing records
- `delete` - Remove records
- `manage` - Full control (all actions)

### Conditions (Advanced)

Conditions allow fine-grained control:

```javascript
{
  "resource": "data",
  "actions": ["read", "update"],
  "conditions": {
    // Only specific collections
    "collection": ["orders", "customers"],
    
    // Only records owned by user
    "owner": "$userId",
    
    // Only specific status
    "status": ["pending", "in_progress"],
    
    // Custom field matching
    "restaurantId": "$userRestaurantId"
  }
}
```

---

## Best Practices

### 1. **Separation of Concerns**
- **System roles** control platform access (owner, admin, developer, viewer)
- **Custom roles** control application logic (customer, seller, rider, etc.)

### 2. **Principle of Least Privilege**
- Grant only necessary permissions
- Start restrictive, expand as needed

### 3. **Role Hierarchy**
```
100 - Platform Owner
 90 - Super Admin
 80 - System Admin
 60 - Developer
 50 - Support Agent
 40 - Restaurant Partner
 30 - Rider
 20 - Viewer
 10 - Customer
```

### 4. **Naming Conventions**
- Use descriptive names: "Support Agent" not "Role1"
- Use snake_case for slugs: "support_agent"
- Keep names consistent across your app

### 5. **Testing Permissions**
Always test with different role assignments:
```javascript
// Get user permissions before deployment
GET /api/orgs/{orgId}/users/{userId}/permissions
```

---

## Migration from Simple Roles

If you're currently using only system roles:

```javascript
// Before: User only has system role
{
  "_id": "user123",
  "role": "developer",
  "customRole": null
}

// After: Add application role
POST /api/orgs/{orgId}/users/user123/role
{
  "roleSlug": "support_agent"
}

// Now user has both
{
  "_id": "user123",
  "role": "developer",          // Platform access
  "customRole": "support_agent"  // Application access
}
```

---

## FAQ

**Q: Can a user have multiple custom roles?**  
A: Currently, users can have ONE system role + ONE custom role. For multi-role needs, create a combined role with all necessary permissions.

**Q: How do I handle role inheritance?**  
A: Use the `hierarchy` field. Higher hierarchy roles can manage lower hierarchy roles.

**Q: Can I modify system roles?**  
A: No, system roles (owner, admin, developer, viewer) cannot be modified. Create custom roles for application-specific needs.

**Q: What happens if I delete a role that users have?**  
A: The API will prevent deletion if any users have that role. Reassign users first.

**Q: How do permissions combine?**  
A: Custom role permissions ADD to system role permissions (union, not override).

**Q: Can roles be project-specific?**  
A: Roles are org-level but permissions can be scoped to specific collections or conditions.

---

## Summary

**MUDBASE's RBAC system fully supports:**
- ✅ Multi-role applications (e-commerce, delivery apps, SaaS)
- ✅ Custom role creation with granular permissions
- ✅ Role hierarchy and inheritance
- ✅ Permission checking at API level
- ✅ Safe role management (prevents deletion of assigned roles)
- ✅ Combined system + custom role permissions
- ✅ Conditional permissions (field-level, ownership-based)

**Perfect for:**
- 🛒 E-commerce platforms (customer, seller, admin)
- 🚗 Delivery apps (customer, rider, restaurant, support)
- 💼 Multi-tenant SaaS (org owner, project admin, team member)
- 🏥 Healthcare apps (patient, doctor, nurse, admin)
- 🎓 Education platforms (student, teacher, parent, admin)

---

## Multi-Role Feature - Ready-to-Use System (NEW)

**Endpoints**: `/api/projects/:projectId/multi-role`, `/api/auth/{method}/signup/{role}`  
**Use Case**: Simplified multi-role system with pre-built templates and role-based signup endpoints

### Overview

The **Multi-Role Feature** is a ready-to-use system (similar to the chat messaging feature) that provides:

- ✅ **Pre-built Role Templates**: 8 default roles (Owner, Super Admin, Admin, Support, Seller, Vendor, Rider, Customer)
- ✅ **Toggle Roles On/Off**: Enable/disable roles per project from dashboard
- ✅ **Custom Roles**: Add your own custom roles
- ✅ **Role-Based Signup**: Role specified in URL path (e.g., `/signup/customer`, `/signup/vendor`)
- ✅ **Collection Permissions**: Configure role permissions per collection
- ✅ **Dashboard Configuration**: All settings managed from project dashboard
- ✅ **All Auth Methods**: Works with Local, OAuth, Magic Link, OTP, Anonymous

### Quick Start

#### 1. Initialize Multi-Role Feature

The feature auto-initializes when first accessed:

```javascript
GET /api/projects/{projectId}/multi-role
Authorization: Bearer {token}

// Response - Pre-built roles ready to use!
{
  "success": true,
  "data": {
    "isEnabled": true,
    "defaultRole": "customer",
    "roles": [
      {
        "slug": "customer",
        "name": "Customer",
        "signupEndpoint": "customer",
        "isEnabled": true,
        "hierarchy": 10
      },
      {
        "slug": "vendor",
        "name": "Vendor",
        "signupEndpoint": "vendor",
        "isEnabled": true,
        "hierarchy": 40,
        "requiresPayment": true,
        "requiresKYC": true
      },
      // ... more roles
    ]
  }
}
```

#### 2. Use Role-Based Signup Endpoints

**Local Auth:**
```javascript
POST /api/auth/local/signup/customer
{
  "email": "customer@example.com",
  "password": "SecurePass123!",
  "firstName": "John",
  "lastName": "Doe",
  "projectId": "507f1f77bcf86cd799439011"
}

// ✅ Role automatically assigned!
```

**OAuth:**
```javascript
GET /api/auth/oauth/signup/vendor/google/{projectId}
// Redirects to Google OAuth, assigns vendor role
```

**Magic Link:**
```javascript
POST /api/auth/magic-link/signup/seller
{
  "email": "seller@example.com",
  "projectId": "507f1f77bcf86cd799439011"
}
```

#### 3. Configure Roles from Dashboard

```javascript
// Toggle role on/off
PATCH /api/projects/{projectId}/multi-role/roles/vendor/toggle
{
  "isEnabled": true
}

// Update role settings
PATCH /api/projects/{projectId}/multi-role/roles/vendor
{
  "requiresApproval": false,
  "requiresPayment": true,
  "requiresKYC": true
}

// Configure collection permissions
PATCH /api/projects/{projectId}/multi-role/roles/vendor/collections/{productsCollectionId}/permissions
{
  "actions": ["create", "read", "update", "delete"],
  "conditions": {
    "vendorId": "$user.metadata.vendorId"
  }
}
```

### Default Role Templates

| Role | Slug | Endpoint | Hierarchy | Requires Approval |
|------|------|----------|-----------|-------------------|
| Owner | `owner` | `/signup/owner` | 100 | ✅ Yes |
| Super Admin | `super_admin` | `/signup/super-admin` | 90 | ✅ Yes |
| Admin | `admin` | `/signup/admin` | 80 | ✅ Yes |
| Support | `support` | `/signup/support` | 50 | ✅ Yes |
| Seller | `seller` | `/signup/seller` | 40 | ❌ No |
| Vendor | `vendor` | `/signup/vendor` | 40 | ❌ No |
| Rider | `rider` | `/signup/rider` | 30 | ❌ No |
| Customer | `customer` | `/signup/customer` | 10 | ❌ No |

### API Endpoints

#### Get Configuration
```javascript
GET /api/projects/{projectId}/multi-role
```

#### Update Settings
```javascript
PATCH /api/projects/{projectId}/multi-role/settings
{
  "isEnabled": true,
  "defaultRole": "customer",
  "settings": {
    "allowMultipleRoles": false,
    "requireRoleSelection": true
  }
}
```

#### Toggle Role
```javascript
PATCH /api/projects/{projectId}/multi-role/roles/{roleSlug}/toggle
{
  "isEnabled": true
}
```

#### Update Role
```javascript
PATCH /api/projects/{projectId}/multi-role/roles/{roleSlug}
{
  "name": "Updated Name",
  "requiresApproval": true,
  "requiresPayment": false
}
```

#### Add Custom Role
```javascript
POST /api/projects/{projectId}/multi-role/roles
{
  "slug": "premium_customer",
  "name": "Premium Customer",
  "signupEndpoint": "premium-customer",
  "hierarchy": 20,
  "defaultPermissions": [...]
}
```

#### Update Collection Permissions
```javascript
PATCH /api/projects/{projectId}/multi-role/roles/{roleSlug}/collections/{collectionId}/permissions
{
  "actions": ["create", "read", "update", "delete"],
  "conditions": {
    "userId": "$userId"
  }
}
```

### Benefits

1. **Ready-to-Use**: Pre-built templates, no coding required
2. **Simple Signup**: Role in URL path, no complex workflows
3. **Dashboard Configuration**: All settings in web interface
4. **Flexible**: Toggle roles, add custom roles
5. **Secure**: Server-side role assignment only
6. **Integrated**: Works with all authentication methods

### Comparison: Multi-Role Feature vs Role Elevation

| Feature | Multi-Role Feature | Role Elevation |
|---------|-------------------|----------------|
| **Use Case** | Initial signup with role | Upgrading existing user's role |
| **Complexity** | Simple, ready-to-use | Configurable, approval workflows |
| **Signup** | Role in URL path | Default role, then elevate |
| **Configuration** | Dashboard UI | API endpoints |
| **Best For** | New projects, simple apps | Complex workflows, payment-based upgrades |

**Recommendation**: Use **Multi-Role Feature** for new projects and simple use cases. Use **Role Elevation** for upgrading existing users or complex approval workflows.

---

## Anonymous Authentication

**Endpoint**: `/api/auth/anonymous`  
**Use Case**: Guest checkout, trial experiences, gaming, content apps

### 1. Create Anonymous Session

```javascript
POST /api/auth/anonymous
Content-Type: application/json

{
  "deviceId": "device-uuid-123" // Optional device identifier
}

// Response
{
  "message": "Anonymous session created",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439050",
    "isAnonymous": true,
    "anonymousId": "anon-abc123",
    "role": "viewer",
    "createdAt": "2024-12-16T10:00:00Z"
  }
}
```

### 2. Use Anonymous Session

```javascript
// Make API calls with anonymous token
GET /api/projects/507f1f77bcf86cd799439011/data/products
Authorization: Bearer {anonymous_token}

// Limited access based on permissions
{
  "data": [
    { "_id": "1", "name": "Product 1", "price": 100 }
  ]
}
```

### 3. Convert to Full Account

```javascript
POST /api/auth/anonymous/convert
Authorization: Bearer {anonymous_token}
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePassword123!",
  "firstName": "John",
  "lastName": "Doe"
}

// Response - Same user ID preserved!
{
  "message": "Anonymous account converted successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439050", // Same ID
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "isAnonymous": false,
    "role": "developer"
  }
}
```

**Benefits:**
- Preserves cart, preferences, progress
- Seamless guest-to-user transition
- Improves conversion rates
- No forced registration

---

## OAuth Account Linking

**Endpoint**: `/api/users/me/oauth-providers`  
**Use Case**: Multiple login options for single account

### 1. List Linked Providers

```javascript
GET /api/users/me/oauth-providers
Authorization: Bearer {token}

// Response
{
  "providers": [
    {
      "provider": "google",
      "providerId": "google-user-id-123",
      "email": "user@gmail.com",
      "linkedAt": "2024-12-01T10:00:00Z"
    },
    {
      "provider": "github",
      "providerId": "github-user-id-456",
      "username": "johndoe",
      "linkedAt": "2024-12-15T14:30:00Z"
    }
  ]
}
```

### 2. Link New OAuth Provider

```javascript
// Redirects to OAuth provider for authorization
GET /api/users/me/oauth-providers/link/google?projectId=507f1f77bcf86cd799439011
Authorization: Bearer {token}

// After user authorizes:
// - Google account linked to existing account
// - User can now login with Google OR email/password

// Response (after callback)
{
  "message": "OAuth provider linked successfully",
  "provider": "google",
  "email": "user@gmail.com"
}
```

### 3. Unlink OAuth Provider

```javascript
DELETE /api/users/me/oauth-providers/github
Authorization: Bearer {token}

// Response
{
  "message": "OAuth provider unlinked successfully",
  "provider": "github"
}

// Note: Cannot unlink if it's the only authentication method
```

**Benefits:**
- Single account, multiple login options
- Prevents duplicate accounts
- Easier account recovery
- Improved UX

---

## GDPR Compliance Tools

**Endpoints**: `/api/users/me/export`, `/api/users/me/erase`  
**Compliance**: GDPR Articles 15 & 17

### 1. Export User Data (Article 15 - Data Portability)

```javascript
GET /api/users/me/export
Authorization: Bearer {token}

// Response - Complete JSON export
{
  "exportedAt": "2024-12-16T10:00:00Z",
  "user": {
    "_id": "507f1f77bcf86cd799439012",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "developer",
    "org": "507f1f77bcf86cd799439013",
    "createdAt": "2024-01-15T08:30:00Z",
    "lastLogin": "2024-12-16T09:00:00Z"
  },
  "projects": [
    {
      "_id": "507f1f77bcf86cd799439011",
      "name": "My App",
      "createdAt": "2024-01-20T10:00:00Z"
    }
  ],
  "wallets": [
    {
      "currency": "BTC",
      "address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
      "balance": "0.005",
      "createdAt": "2024-02-01T12:00:00Z"
    }
  ],
  "transactions": [
    // All wallet transactions
  ],
  "files": [
    // File metadata (not file contents for size reasons)
  ],
  "integrations": [
    // All configured integrations
  ],
  "apiKeys": [
    {
      "name": "Production Key",
      "createdAt": "2024-03-01T10:00:00Z",
      "lastUsed": "2024-12-16T08:00:00Z"
      // Note: Actual keys not exposed for security
    }
  ]
}
```

### 2. Delete User Data (Article 17 - Right to Be Forgotten)

```javascript
POST /api/users/me/erase
Authorization: Bearer {token}
Content-Type: application/json

{
  "confirmation": "DELETE_MY_ACCOUNT",
  "reason": "No longer need the service" // Optional
}

// Response
{
  "message": "Account erasure initiated",
  "details": {
    "dataAnonymized": true,
    "apiKeysDeactivated": true,
    "sessionsDeleted": true,
    "accountDisabled": true,
    "scheduledForDeletion": "2024-12-23T10:00:00Z" // 7-day grace period
  }
}
```

**What Gets Erased:**
- ✅ Email → `deleted-user-{hash}@deleted.local`
- ✅ Name → `Deleted User`
- ✅ All PII removed or anonymized
- ✅ API keys deactivated
- ✅ Sessions terminated
- ✅ Account marked inactive

**What's Preserved (Legal/Financial):**
- Transaction records (anonymized)
- Audit logs (anonymized)
- Billing history (required for accounting)

**Grace Period:**
- 7 days before final erasure
- User can cancel during grace period

---

## Stripe Invoicing

**Endpoint**: `/api/billing/projects/:projectId/invoices`  
**Use Case**: Professional invoicing, accounting, tax compliance

### 1. List Invoices

```javascript
GET /api/billing/projects/507f1f77bcf86cd799439011/invoices?limit=10&status=paid
Authorization: Bearer {token}

// Response
{
  "invoices": [
    {
      "_id": "507f1f77bcf86cd799439060",
      "invoiceNumber": "INV-2024-001",
      "stripeInvoiceId": "in_1ABC123",
      "subscription": "507f1f77bcf86cd799439015",
      "project": "507f1f77bcf86cd799439011",
      "org": "507f1f77bcf86cd799439013",
      "customer": {
        "name": "Acme Corp",
        "email": "billing@acme.com",
        "address": "123 Main St, City, State, ZIP"
      },
      "status": "paid",
      "currency": "usd",
      "subtotal": 2900,  // $29.00
      "tax": 290,        // $2.90
      "total": 3190,     // $31.90
      "amountPaid": 3190,
      "amountDue": 0,
      "lineItems": [
        {
          "description": "Pro Plan - Monthly",
          "quantity": 1,
          "unitAmount": 2900,
          "amount": 2900
        }
      ],
      "dueDate": "2024-12-01T00:00:00Z",
      "paidAt": "2024-12-01T10:30:00Z",
      "hostedInvoiceUrl": "https://invoice.stripe.com/...",
      "invoicePdfUrl": "https://invoice.stripe.com/.../pdf",
      "createdAt": "2024-12-01T00:00:00Z"
    }
  ],
  "total": 12,
  "page": 1,
  "limit": 10
}
```

### 2. Get Single Invoice

```javascript
GET /api/billing/projects/507f1f77bcf86cd799439011/invoices/507f1f77bcf86cd799439060
Authorization: Bearer {token}

// Response - Same structure as above, single invoice
```

### 3. Download Invoice PDF

```javascript
GET /api/billing/projects/507f1f77bcf86cd799439011/invoices/507f1f77bcf86cd799439060/download
Authorization: Bearer {token}

// Redirects to Stripe hosted PDF or streams PDF directly
// Response: PDF file download
```

### 4. Export Invoice as JSON

```javascript
GET /api/billing/projects/507f1f77bcf86cd799439011/invoices/507f1f77bcf86cd799439060/export
Authorization: Bearer {token}

// Response - Accounting-friendly JSON
{
  "invoice": {
    "invoiceNumber": "INV-2024-001",
    "issueDate": "2024-12-01T00:00:00Z",
    "dueDate": "2024-12-01T00:00:00Z",
    "paidDate": "2024-12-01T10:30:00Z",
    "customer": {
      "name": "Acme Corp",
      "email": "billing@acme.com",
      "taxId": "US123456789"
    },
    "lineItems": [
      {
        "description": "Pro Plan - Monthly",
        "quantity": 1,
        "unitPrice": 29.00,
        "amount": 29.00
      }
    ],
    "subtotal": 29.00,
    "tax": 2.90,
    "total": 31.90,
    "currency": "USD",
    "status": "paid",
    "paymentMethod": "card_****1234"
  }
}
```

**Features:**
- Automatic invoice generation
- PDF download
- JSON export for accounting systems
- Tax calculation
- Payment status tracking
- Hosted invoice page

---

## Row-Level Security (RLS)

**Implementation**: Transparent middleware  
**Scope**: All database queries automatically filtered

### How It Works

Every authenticated request includes organization and project context. All database queries are automatically scoped:

```javascript
// Developer makes request:
GET /api/projects/507f1f77bcf86cd799439011/data/products
Authorization: Bearer {token}

// Behind the scenes, middleware adds filters:
{
  org: "507f1f77bcf86cd799439013",      // From user's token
  project: "507f1f77bcf86cd799439011",   // From URL parameter
  // Additional scoping based on endpoint
}

// Actual MongoDB query:
db.products.find({
  org: "507f1f77bcf86cd799439013",
  project: "507f1f77bcf86cd799439011"
})
```

### Protected Resources

RLS automatically applied to:
- ✅ All project data (collections, documents)
- ✅ File storage (buckets, files)
- ✅ Wallets and transactions
- ✅ Billing and subscriptions
- ✅ Integrations
- ✅ Webhooks
- ✅ Usage statistics
- ✅ Schemas and collections
- ✅ Search results

### Security Benefits

- **Prevents unauthorized access**: Users can only see their organization's data
- **No manual filtering needed**: Developers don't need to add org/project filters
- **Consistent across platform**: Applied to all sensitive endpoints
- **Reduces developer errors**: Can't accidentally expose other users' data

### Implementation Files

- `middleware/rls.js` - RLS middleware
- `utils/rlsHelper.js` - Helper functions for controllers
- Applied in: All 15+ sensitive controllers

---

## Backup & Restore

**Endpoint**: `/api/projects/:projectId/backups`  
**Use Case**: Disaster recovery, testing, migrations

### 1. Create Backup

```javascript
POST /api/projects/507f1f77bcf86cd799439011/backups
Authorization: Bearer {token}
Content-Type: application/json

{
  "description": "Pre-deployment backup",
  "includeFiles": true,        // Include file storage
  "includeWallets": false       // Exclude wallets for security
}

// Response
{
  "backup": {
    "_id": "507f1f77bcf86cd799439070",
    "project": "507f1f77bcf86cd799439011",
    "description": "Pre-deployment backup",
    "status": "in_progress",
    "size": 0,
    "collections": [],
    "createdBy": "507f1f77bcf86cd799439012",
    "createdAt": "2024-12-16T10:00:00Z",
    "estimatedCompletion": "2024-12-16T10:05:00Z"
  }
}
```

### 2. List Backups

```javascript
GET /api/projects/507f1f77bcf86cd799439011/backups
Authorization: Bearer {token}

// Response
{
  "backups": [
    {
      "_id": "507f1f77bcf86cd799439070",
      "description": "Pre-deployment backup",
      "status": "completed",
      "size": 1048576, // bytes
      "collections": ["products", "users", "orders"],
      "fileCount": 150,
      "createdAt": "2024-12-16T10:00:00Z",
      "completedAt": "2024-12-16T10:04:32Z"
    },
    {
      "_id": "507f1f77bcf86cd799439071",
      "description": "Automatic daily backup",
      "status": "completed",
      "size": 1122334,
      "createdAt": "2024-12-15T02:00:00Z"
    }
  ],
  "total": 5
}
```

### 3. Restore from Backup

```javascript
POST /api/projects/507f1f77bcf86cd799439011/backups/507f1f77bcf86cd799439070/restore
Authorization: Bearer {token}
Content-Type: application/json

{
  "restoreMode": "replace",           // or "merge"
  "collections": ["products", "orders"], // Optional: specific collections only
  "confirmation": "RESTORE_DATA"
}

// Response
{
  "message": "Restore initiated",
  "restore": {
    "_id": "507f1f77bcf86cd799439071",
    "backup": "507f1f77bcf86cd799439070",
    "status": "in_progress",
    "restoreMode": "replace",
    "collectionsToRestore": ["products", "orders"],
    "startedAt": "2024-12-16T11:00:00Z",
    "estimatedCompletion": "2024-12-16T11:10:00Z"
  }
}
```

### 4. Delete Backup

```javascript
DELETE /api/projects/507f1f77bcf86cd799439011/backups/507f1f77bcf86cd799439070
Authorization: Bearer {token}

// Response
{
  "message": "Backup deleted successfully",
  "backup": {
    "_id": "507f1f77bcf86cd799439070",
    "size": 1048576,
    "deletedAt": "2024-12-16T11:30:00Z"
  }
}
```

### Restore Modes

**Replace Mode:**
- Deletes existing data
- Restores from backup
- Use for: Rolling back bad deployments

**Merge Mode:**
- Keeps existing data
- Adds data from backup
- Overwrites conflicts (based on _id)
- Use for: Recovering deleted records

### Backup Features

- ✅ Manual backups on-demand
- ✅ Automatic daily backups (Enterprise plan)
- ✅ Point-in-time recovery
- ✅ Selective collection restore
- ✅ Replace or merge modes
- ✅ Backup retention policies (30 days default)
- ✅ Encrypted storage
- ✅ Progress tracking

---

## Virus Scanning

**Implementation**: Automatic on all file uploads  
**Providers**: ClamAV, VirusTotal, Mock (testing)

### Upload with Automatic Scanning

```javascript
// Regular file upload - scanning happens automatically
POST /api/files/507f1f77bcf86cd799439011/upload
Authorization: Bearer {token}
Content-Type: multipart/form-data

{
  file: [binary data],
  bucket: "images",
  isPublic: true
}

// Response if file is clean
{
  "file": {
    "_id": "abc123",
    "originalName": "photo.jpg",
    "fileName": "507f1f77bcf86cd799439011/images/abc123-photo.jpg",
    "mimeType": "image/jpeg",
    "size": 1048576,
    "url": "https://cdn.mudbase.io/...",
    "metadata": {
      "virusScan": {
        "status": "clean",
        "provider": "clamav",
        "scannedAt": "2024-12-16T10:00:00Z",
        "hash": "sha256:abc123def456..."
      }
    }
  }
}

// Response if malware detected
{
  "error": "File rejected: Win32.Trojan.Generic",
  "code": "MALWARE_DETECTED",
  "details": {
    "threat": "Win32.Trojan.Generic",
    "provider": "clamav",
    "action": "rejected"
  }
}
```

### Configuration

**Environment Variables:**

```bash
# Enable/disable scanning
VIRUS_SCAN_ENABLED=true

# Choose provider
VIRUS_SCAN_PROVIDER=clamav  # or "virustotal" or "mock"

# Strict mode: reject file on scan failure
VIRUS_SCAN_STRICT=true

# ClamAV configuration
CLAMAV_HOST=localhost
CLAMAV_PORT=3310

# VirusTotal configuration
VIRUSTOTAL_API_KEY=your-api-key

# Mock (development only)
MOCK_BLOCK_EXECUTABLES=true
```

### Scanning Providers

**1. ClamAV (Recommended for Production)**
- Self-hosted, free, open-source
- Real-time scanning
- Regular signature updates
- Low latency

**2. VirusTotal (Cloud-based)**
- 70+ antivirus scanners
- Higher accuracy
- API rate limits apply
- Slower (network latency)
- Requires API key

**3. Mock (Development Only)**
- No external dependencies
- Detects EICAR test file only
- Use for testing/development

### Features

- ✅ Automatic scanning on upload
- ✅ File rejection on malware detection
- ✅ Scan metadata stored with files
- ✅ Multiple provider support
- ✅ Configurable strict mode
- ✅ Hash-based tracking
- ✅ Quarantine workflow

---

## Per-File RBAC

**Endpoint**: `/api/files/:fileId/acl`  
**Use Case**: Sensitive documents, shared files, temporary access

### 1. Set File Permissions

```javascript
PATCH /api/files/abc123/acl
Authorization: Bearer {token}
Content-Type: application/json

{
  "acl": [
    {
      "user": "507f1f77bcf86cd799439012",
      "permissions": ["read", "write", "delete"]
    },
    {
      "role": "developer",
      "permissions": ["read", "write"]
    },
    {
      "role": "viewer",
      "permissions": ["read"]
    },
    {
      "group": "marketing-team",
      "permissions": ["read"]
    }
  ],
  "passwordProtected": true,
  "password": "SecurePass123!",
  "expiresAt": "2024-12-31T23:59:59Z",
  "downloadLimit": 10
}

// Response
{
  "message": "File permissions updated",
  "file": {
    "_id": "abc123",
    "originalName": "confidential.pdf",
    "acl": [
      {
        "user": "507f1f77bcf86cd799439012",
        "permissions": ["read", "write", "delete"]
      },
      {
        "role": "developer",
        "permissions": ["read", "write"]
      }
    ],
    "isPasswordProtected": true,
    "expiresAt": "2024-12-31T23:59:59Z",
    "downloadLimit": 10,
    "downloadCount": 0
  }
}
```

### 2. Access Password-Protected File

```javascript
// Download file with password
GET /api/files/abc123/download
Authorization: Bearer {token}
X-File-Password: SecurePass123!

// Response: File download (200) or Forbidden (403)
```

### 3. Check File Permissions

```javascript
GET /api/files/abc123/acl
Authorization: Bearer {token}

// Response
{
  "acl": [
    {
      "user": "507f1f77bcf86cd799439012",
      "permissions": ["read", "write", "delete"]
    },
    {
      "role": "developer",
      "permissions": ["read", "write"]
    }
  ],
  "hasAccess": true,
  "yourPermissions": ["read", "write"],
  "isPasswordProtected": true,
  "downloadLimit": 10,
  "downloadCount": 3,
  "remainingDownloads": 7,
  "expiresAt": "2024-12-31T23:59:59Z",
  "isExpired": false
}
```

### 4. Remove File Permissions

```javascript
DELETE /api/files/abc123/acl
Authorization: Bearer {token}

// Response
{
  "message": "File permissions removed",
  "file": {
    "_id": "abc123",
    "acl": [], // Empty - back to project-level permissions
    "isPasswordProtected": false
  }
}
```

### Permission Types

- **read** - View/download file
- **write** - Modify file metadata
- **delete** - Delete file

### Access Control Options

1. **User-specific**: Grant permissions to specific users
2. **Role-based**: Grant permissions based on user role (owner, admin, developer, viewer)
3. **Group-based**: Grant permissions to groups/teams
4. **Password protection**: Require password for access
5. **Download limits**: Limit number of downloads
6. **Expiration dates**: Auto-expire access

### Use Cases

- **Confidential documents**: Limit access to specific users
- **Temporary sharing**: Share with expiration date
- **Client deliverables**: Limit downloads to prevent over-distribution
- **Sensitive data**: Add password protection layer
- **Team collaboration**: Grant access to specific groups

---

## Real-time Analytics

**Endpoint**: `/api/realtime/projects/:projectId/*`  
**Use Case**: Live dashboards, monitoring, user presence

### 1. Get Project Analytics

```javascript
GET /api/realtime/projects/507f1f77bcf86cd799439011/analytics
Authorization: Bearer {token}

// Response
{
  "projectId": "507f1f77bcf86cd799439011",
  "activeConnections": 42,
  "totalEvents": 15234,
  "lastActivity": "2024-12-16T10:05:30Z",
  "timestamp": "2024-12-16T10:05:32Z"
}
```

### 2. Get Active Users

```javascript
GET /api/realtime/projects/507f1f77bcf86cd799439011/active-users
Authorization: Bearer {token}

// Response
{
  "users": [
    {
      "userId": "507f1f77bcf86cd799439012",
      "connectedAt": "2024-12-16T09:30:00Z",
      "socketId": "socket-abc123"
    },
    {
      "userId": "507f1f77bcf86cd799439013",
      "connectedAt": "2024-12-16T10:00:00Z",
      "socketId": "socket-def456"
    }
  ],
  "count": 2,
  "timestamp": "2024-12-16T10:05:32Z"
}
```

### 3. Check User Presence

```javascript
POST /api/realtime/projects/507f1f77bcf86cd799439011/presence
Authorization: Bearer {token}
Content-Type: application/json

{
  "userIds": [
    "507f1f77bcf86cd799439012",
    "507f1f77bcf86cd799439013",
    "507f1f77bcf86cd799439014"
  ]
}

// Response
{
  "presence": {
    "507f1f77bcf86cd799439012": {
      "online": true,
      "lastSeen": "2024-12-16T10:05:00Z"
    },
    "507f1f77bcf86cd799439013": {
      "online": true,
      "lastSeen": "2024-12-16T10:04:30Z"
    },
    "507f1f77bcf86cd799439014": {
      "online": false,
      "lastSeen": "2024-12-15T18:30:00Z"
    }
  },
  "timestamp": "2024-12-16T10:05:32Z"
}
```

### 4. Get Event Throughput

```javascript
GET /api/realtime/projects/507f1f77bcf86cd799439011/throughput?window=60000
Authorization: Bearer {token}

// Response
{
  "windowMs": 60000,
  "totalEvents": 523,
  "eventsPerSecond": 8.72,
  "byType": {
    "db_create": 150,
    "db_update": 200,
    "db_delete": 50,
    "project_subscribe": 100,
    "project_unsubscribe": 23
  },
  "timestamp": "2024-12-16T10:05:32Z"
}
```

### 5. Get Historical Analytics

```javascript
GET /api/realtime/projects/507f1f77bcf86cd799439011/history?period=hour
Authorization: Bearer {token}

// Response
{
  "projectId": "507f1f77bcf86cd799439011",
  "period": "hour",
  "data": [
    {
      "timestamp": "2024-12-16T09:00:00Z",
      "connections": 35,
      "events": 420
    },
    {
      "timestamp": "2024-12-16T09:01:00Z",
      "connections": 37,
      "events": 445
    },
    {
      "timestamp": "2024-12-16T09:02:00Z",
      "connections": 39,
      "events": 468
    }
    // ... 60 data points (one per minute for past hour)
  ],
  "generatedAt": "2024-12-16T10:05:32Z"
}
```

### Metrics Tracked

- **Active Connections**: Current Socket.IO connections per project
- **Active Users**: Unique users currently connected
- **Event Throughput**: Events per second, broken down by type
- **User Presence**: Online/offline status with last seen timestamp
- **Historical Data**: Time-series data for visualization

### Use Cases

- **Live dashboards**: Show real-time user activity
- **Monitoring**: Track system health and usage
- **Presence indicators**: "Who's online" features
- **Analytics**: Usage patterns and trends
- **Capacity planning**: Peak usage times

### Integration with Socket.IO

Real-time analytics automatically track:
- Socket.IO connections/disconnections
- Project subscriptions
- Database events (create/update/delete)
- Custom events

No additional code needed - tracking is transparent!

---

## Webhook Transformations

**Endpoint**: `/api/webhooks/projects/:projectId/config`  
**Use Case**: Payload customization, filtering, versioning

### 1. Configure Transformations

```javascript
PUT /api/webhooks/projects/507f1f77bcf86cd799439011/config
Authorization: Bearer {token}
Content-Type: application/json

{
  "transformations": [
    {
      "type": "fieldMapping",
      "config": {
        "data.userId": "user_id",
        "data.timestamp": "created_at",
        "data.orderTotal": "amount"
      }
    },
    {
      "type": "filter",
      "config": {
        "condition": "data.amount > 100"
      }
    },
    {
      "type": "version",
      "config": {
        "version": "v2"
      }
    }
  ]
}

// Response
{
  "message": "Webhook configuration updated",
  "config": {
    "transformations": [
      {
        "type": "fieldMapping",
        "config": {
          "data.userId": "user_id",
          "data.timestamp": "created_at",
          "data.orderTotal": "amount"
        }
      },
      {
        "type": "filter",
        "config": {
          "condition": "data.amount > 100"
        }
      },
      {
        "type": "version",
        "config": {
          "version": "v2"
        }
      }
    ]
  }
}
```

### 2. Test Transformation

```javascript
POST /api/webhooks/projects/507f1f77bcf86cd799439011/test-transformation
Authorization: Bearer {token}
Content-Type: application/json

{
  "payload": {
    "event": "order.created",
    "data": {
      "userId": "user123",
      "orderTotal": 150.50,
      "timestamp": "2024-12-16T10:00:00Z"
    }
  },
  "transformations": [
    {
      "type": "fieldMapping",
      "config": {
        "data.userId": "user_id",
        "data.orderTotal": "amount"
      }
    }
  ]
}

// Response
{
  "original": {
    "event": "order.created",
    "data": {
      "userId": "user123",
      "orderTotal": 150.50,
      "timestamp": "2024-12-16T10:00:00Z"
    }
  },
  "transformed": {
    "event": "order.created",
    "data": {
      "user_id": "user123",
      "amount": 150.50,
      "timestamp": "2024-12-16T10:00:00Z"
    }
  }
}
```

### 3. Get Webhook Configuration

```javascript
GET /api/webhooks/projects/507f1f77bcf86cd799439011/config
Authorization: Bearer {token}

// Response
{
  "config": {
    "transformations": [
      {
        "type": "fieldMapping",
        "config": {
          "data.userId": "user_id"
        }
      }
    ],
    "version": "v2"
  }
}
```

### Transformation Types

#### 1. Field Mapping

Rename fields in the payload:

```json
{
  "type": "fieldMapping",
  "config": {
    "data.userId": "user_id",
    "data.createdAt": "timestamp"
  }
}
```

#### 2. Filtering

Only send webhooks that match conditions:

```json
{
  "type": "filter",
  "config": {
    "condition": "data.amount > 100 && data.status === 'completed'"
  }
}
```

#### 3. Versioning

Wrap payload with version info:

```json
{
  "type": "version",
  "config": {
    "version": "v2"
  }
}
```

**Result:**
```json
{
  "version": "v2",
  "payload": {
    /* original payload */
  }
}
```

#### 4. Flattening

Flatten nested objects:

```json
{
  "type": "flatten",
  "config": {
    "separator": "_"
  }
}
```

**Before:**
```json
{
  "data": {
    "user": {
      "id": "123",
      "name": "John"
    }
  }
}
```

**After:**
```json
{
  "data_user_id": "123",
  "data_user_name": "John"
}
```

#### 5. Template

Use templates for custom output:

```json
{
  "type": "template",
  "config": {
    "template": {
      "notification": "New order #{{data.orderId}} from {{data.customer.name}}",
      "amount": "{{data.amount}}"
    }
  }
}
```

#### 6. JSON Path

Complex transformations using JSON path:

```json
{
  "type": "jsonpath",
  "config": {
    "expressions": {
      "$.data.items[*].price": "$.prices"
    }
  }
}
```

### Example: Complete Transformation

**Original Payload:**
```json
{
  "event": "order.created",
  "data": {
    "userId": "user123",
    "items": [{...}],
    "total": 150.50,
    "createdAt": "2024-12-16T10:00:00Z"
  }
}
```

**After Transformations:**
```json
{
  "version": "v2",
  "event": "order.created",
  "payload": {
    "user_id": "user123",
    "items": [{...}],
    "amount": 150.50,
    "timestamp": "2024-12-16T10:00:00Z"
  },
  "metadata": {
    "transformedAt": "2024-12-16T10:00:01Z",
    "version": "v2"
  }
}
```

---

## Compliance Automation

**Endpoint**: `/api/compliance/*`  
**Use Case**: SOC 2, GDPR, HIPAA compliance evidence

### 1. Access Review Report (SOC 2)

```javascript
POST /api/compliance/access-review
Authorization: Bearer {token}
Content-Type: application/json

{
  "orgId": "507f1f77bcf86cd799439013",
  "reviewPeriod": {
    "start": "2024-10-01T00:00:00Z",
    "end": "2024-12-31T23:59:59Z"
  }
}

// Response
{
  "report": {
    "orgId": "507f1f77bcf86cd799439013",
    "reviewPeriod": {
      "start": "2024-10-01T00:00:00Z",
      "end": "2024-12-31T23:59:59Z"
    },
    "users": [
      {
        "userId": "507f1f77bcf86cd799439012",
        "email": "john@acme.com",
        "role": "owner",
        "lastLogin": "2024-12-16T09:00:00Z",
        "mfaEnabled": true,
        "apiKeys": 2,
        "projects": 5
      },
      {
        "userId": "507f1f77bcf86cd799439013",
        "email": "jane@acme.com",
        "role": "admin",
        "lastLogin": "2024-12-14T15:30:00Z",
        "mfaEnabled": true,
        "apiKeys": 1,
        "projects": 3
      }
    ],
    "summary": {
      "totalUsers": 10,
      "activeUsers": 8,
      "inactiveUsers": 2,
      "mfaAdoptionRate": 80,
      "adminCount": 3,
      "developerCount": 5,
      "viewerCount": 2
    },
    "recommendations": [
      "Enable MFA for remaining 2 users",
      "Review inactive user accounts"
    ],
    "generatedAt": "2024-12-16T10:00:00Z"
  }
}
```

### 2. Data Processing Record (GDPR Article 30)

```javascript
POST /api/compliance/data-processing-record
Authorization: Bearer {token}
Content-Type: application/json

{
  "orgId": "507f1f77bcf86cd799439013",
  "recordDate": "2024-12-16T00:00:00Z"
}

// Response
{
  "record": {
    "orgId": "507f1f77bcf86cd799439013",
    "recordDate": "2024-12-16T00:00:00Z",
    "dataController": {
      "name": "Acme Corp",
      "contact": "dpo@acme.com",
      "address": "123 Main St, City, State, ZIP"
    },
    "dataProtectionOfficer": {
      "name": "John Smith",
      "email": "dpo@acme.com"
    },
    "processingActivities": [
      {
        "purpose": "User authentication and authorization",
        "legalBasis": "Contract",
        "categories": ["Identity data", "Contact data", "Credentials"],
        "recipients": ["Internal systems", "Cloud infrastructure providers"],
        "retention": "Account lifetime + 30 days",
        "securityMeasures": [
          "AES-256 encryption at rest",
          "TLS 1.3 in transit",
          "bcrypt password hashing",
          "Role-based access controls",
          "Multi-factor authentication",
          "Audit logging",
          "Regular security scans"
        ]
      },
      {
        "purpose": "Cryptocurrency wallet management",
        "legalBasis": "Contract",
        "categories": ["Financial data", "Transaction history"],
        "recipients": ["Internal systems", "Blockchain networks"],
        "retention": "Account lifetime + 7 years (legal requirement)",
        "securityMeasures": [
          "AES-256-GCM encryption",
          "Hardware security module integration",
          "Private key encryption",
          "Transaction signing verification"
        ]
      }
    ],
    "dataSubjects": ["End users", "Employees", "Contractors"],
    "internationalTransfers": [
      {
        "country": "United States",
        "safeguards": "Standard Contractual Clauses (SCCs)"
      }
    ],
    "breachNotificationProcedure": "Within 72 hours to supervisory authority",
    "dataSubjectRights": [
      "Right to access",
      "Right to rectification",
      "Right to erasure",
      "Right to data portability",
      "Right to object",
      "Right to restrict processing"
    ],
    "generatedAt": "2024-12-16T10:00:00Z"
  }
}
```

### 3. Get Compliance Summary

```javascript
GET /api/compliance/summary
Authorization: Bearer {token}

// Response
{
  "compliance": {
    "gdpr": {
      "dataExportEnabled": true,
      "dataErasureEnabled": true,
      "consentManagement": true,
      "dataProcessingRecordsAvailable": true,
      "breachNotificationProcedure": true,
      "dpoAppointed": true
    },
    "soc2": {
      "accessReviewsEnabled": true,
      "auditLoggingEnabled": true,
      "encryptionEnabled": true,
      "backupEnabled": true,
      "incidentResponsePlan": true,
      "mfaEnforced": false,
      "lastAccessReview": "2024-12-01T00:00:00Z"
    },
    "security": {
      "passwordPolicy": "strong",
      "sessionTimeout": 1800,
      "rateLimiting": true,
      "virusScanning": true,
      "securityScanning": true,
      "encryptionAtRest": true,
      "encryptionInTransit": true
    },
    "certifications": [],
    "lastAudit": "2024-12-01T00:00:00Z",
    "nextAudit": "2025-03-01T00:00:00Z",
    "auditStatus": "in_progress"
  }
}
```

### 4. Log Security Event

```javascript
POST /api/compliance/security-event
Authorization: Bearer {token}
Content-Type: application/json

{
  "eventType": "unauthorized_access_attempt",
  "severity": "high",
  "details": {
    "userId": "507f1f77bcf86cd799439012",
    "resource": "admin-panel",
    "ipAddress": "192.168.1.100",
    "userAgent": "Mozilla/5.0...",
    "action": "blocked",
    "reason": "Insufficient permissions"
  }
}

// Response
{
  "message": "Security event logged",
  "event": {
    "_id": "507f1f77bcf86cd799439080",
    "eventType": "unauthorized_access_attempt",
    "severity": "high",
    "timestamp": "2024-12-16T10:00:00Z",
    "details": {
      "userId": "507f1f77bcf86cd799439012",
      "resource": "admin-panel",
      "ipAddress": "192.168.1.100",
      "action": "blocked"
    }
  }
}
```

### Security Event Types

- `unauthorized_access_attempt`
- `brute_force_attempt`
- `suspicious_api_activity`
- `private_key_export`
- `bulk_data_export`
- `admin_privilege_escalation`
- `data_breach_detected`
- `mfa_bypass_attempt`

### Compliance Features

- ✅ SOC 2 evidence automation
- ✅ GDPR Article 30 records
- ✅ Access review reports
- ✅ Security event logging
- ✅ Audit trail maintenance
- ✅ Retention policy enforcement
- ✅ Breach notification procedures
- ✅ Data subject rights management

---

## OpenTelemetry Tracing

**Implementation**: Transparent distributed tracing  
**Use Case**: Performance monitoring, debugging, security alerts

### How It Works

Every request automatically gets:
- Unique `trace-id` and `request-id`
- Request span tracking
- Performance measurements
- Error tracking
- Security event alerts

### Request Tracing Example

```javascript
// Make any API request
GET /api/projects/507f1f77bcf86cd799439011/data/products
Authorization: Bearer {token}

// Response includes trace headers
// X-Request-Id: abc-123-def-456
// X-Trace-Id: trace-abc123def456

// Trace data automatically collected:
{
  "traceId": "trace-abc123def456",
  "spans": [
    {
      "spanId": "span-001",
      "name": "HTTP GET /api/projects/.../data/products",
      "startTime": 1702728000000,
      "endTime": 1702728000150,
      "duration": 150, // ms
      "status": "OK",
      "attributes": {
        "http.method": "GET",
        "http.url": "/api/projects/.../data/products",
        "http.status_code": 200,
        "user.id": "507f1f77bcf86cd799439012",
        "project.id": "507f1f77bcf86cd799439011"
      },
      "events": [
        {
          "name": "query_executed",
          "timestamp": 1702728000050,
          "attributes": {
            "collection": "products",
            "duration_ms": 45
          }
        }
      ]
    }
  ]
}
```

### Security Event Alerts

Critical security events trigger automatic alerts:

```javascript
// Example: Private key export triggers alert
POST /api/wallet/projects/507f1f77bcf86cd799439011/wallets/wallet-abc123/export-key
Authorization: Bearer {token}

// Automatically triggers security alert
{
  "alert": {
    "eventType": "private_key_export",
    "severity": "CRITICAL",
    "timestamp": "2024-12-16T10:00:00Z",
    "details": {
      "userId": "507f1f77bcf86cd799439012",
      "walletId": "wallet-abc123",
      "projectId": "507f1f77bcf86cd799439011",
      "ipAddress": "192.168.1.100",
      "userAgent": "Mozilla/5.0..."
    },
    "actions": [
      "logged_to_compliance",
      "sent_to_monitoring",
      "added_to_audit_log"
    ]
  }
}
```

### Monitored Security Events

- **Private key exports** - CRITICAL
- **Bulk data exports** - HIGH
- **Admin privilege escalation** - HIGH
- **Suspicious API activity** - MEDIUM
- **Brute force attempts** - MEDIUM
- **Data breach detection** - CRITICAL

### Tracing Features

- ✅ Automatic request tracing
- ✅ Distributed trace context
- ✅ Span tracking for operations
- ✅ Performance measurement
- ✅ Error tracking
- ✅ Security event alerts
- ✅ Custom event tracking
- ✅ Export to Jaeger/Zipkin/Cloud providers

### Integration

The tracing system integrates with:
- **Compliance service**: Security events logged
- **Monitoring systems**: Metrics and alerts
- **Audit logs**: All traces persisted
- **External APM tools**: Jaeger, Zipkin, Datadog, etc.

---

## Summary

### All New Features (30+)

1. **Multi-Role RBAC** ⭐ - Custom roles with granular permissions
2. **Role Management API** ⭐ - Create/update/delete custom roles
3. **Role Assignment** ⭐ - Assign application roles to users
4. **Permission System** ⭐ - Resource + action-based permissions
5. **Role Hierarchy** ⭐ - Numeric hierarchy for role precedence
6. **Conditional Permissions** ⭐ - Field-level, ownership-based access
7. **Anonymous Authentication** - Guest access with conversion
8. **OAuth Account Linking** - Multi-provider single account
9. **GDPR Data Export** - Complete user data download
10. **GDPR Right to Erasure** - Account deletion/anonymization
11. **Stripe Invoicing** - Professional invoice management
12. **Row-Level Security** - Automatic data scoping
13. **Backup API** - Create project backups
14. **Restore API** - Restore from backups
15. **Virus Scanning** - Automatic malware detection
16. **Per-File RBAC** - Granular file permissions
17. **Real-time Analytics** - Active users, connections
18. **User Presence** - Online/offline tracking
19. **Event Throughput** - Performance metrics
20. **Webhook Transformations** - Payload customization
21. **Field Mapping** - Rename webhook fields
22. **Webhook Filtering** - Conditional delivery
23. **Webhook Versioning** - Version wrapping
24. **Access Review Reports** - SOC 2 compliance
25. **Data Processing Records** - GDPR Article 30
26. **Compliance Summary** - Dashboard data
27. **Security Event Logging** - Audit trail
28. **OpenTelemetry Tracing** - Request tracing
29. **Security Alerts** - Critical event notifications
30. **Distributed Tracing** - Performance monitoring

### Platform Improvements

- **Security & RBAC**: 5/10 → 9.5/10 (+4.5) 🎯
- **Real-time**: 4/10 → 9/10 (+5.0)
- **File Storage**: 3/10 → 8.5/10 (+5.5)
- **Developer Experience**: 6/10 → 9/10 (+3.0)
- **Overall**: 6.8/10 → 8.0/10 (+1.2) 🚀

### Documentation

For more information, see:
- [WORKFLOW_SIMULATION.md](./WORKFLOW_SIMULATION.md) - Core features
- [COMPETITIVE_COMPARISON.md](./COMPETITIVE_COMPARISON.md) - Updated scores
- [COMPLETION_REPORT.md](./COMPLETION_REPORT.md) - Comprehensive report
- [WALLET_ROADMAP.md](./WALLET_ROADMAP.md) - Future wallet features
- [COMPLIANCE_ROADMAP.md](./COMPLIANCE_ROADMAP.md) - SOC 2/GDPR/HIPAA plans
- [FULL_TEXT_SEARCH.md](./FULL_TEXT_SEARCH.md) - Search guide

---

**Last Updated**: December 2024  
**Platform Status**: Production-ready 🚀

