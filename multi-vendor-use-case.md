## 11. Multi-Vendor Application Use Case

### Overview

This use case demonstrates how to build a multi-vendor marketplace application (like Uber Eats, DoorDash, or a local delivery platform) using MUDBASE's project-based authentication and collection schema system. The application supports multiple user types: **customers**, **vendors/sellers**, **riders/delivery drivers**, and **platform admins**.

### Architecture

```
┌────────────────────────────────────────────────────────────┐
│              Multi-Vendor Marketplace Platform             │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Customers  │  │   Vendors    │  │    Riders    │      │
│  │              │  │              │  │              │      │
│  │ • Browse     │  │ • Manage     │  │ • Accept     │      │
│  │ • Order      │  │   Products   │  │   Orders     │      │
│  │ • Track      │  │ • View       │  │ • Update     │      │
│  │              │  │   Orders     │  │   Status     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Platform Admin                         │   │
│  │  • Manage vendors                                   │   │
│  │  • View all orders                                  │   │
│  │  • Analytics & reports                              │   │
│  └─────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────┘
```

### Step 1: Project Setup

```javascript
// Create a new project for the marketplace
POST /api/orgs/{orgId}/projects
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "FoodDelivery Marketplace",
  "description": "Multi-vendor food delivery platform",
  "settings": {
    "frontendUrl": "https://marketplace.example.com"
  },
  "auth": {
    "methods": ["local", "oauth"],
    "providers": ["google", "facebook"]
  }
}

// Response
{
  "_id": "507f1f77bcf86cd799439050",
  "name": "FoodDelivery Marketplace",
  "slug": "fooddelivery-marketplace",
  "org": "507f1f77bcf86cd799439011",
  "auth": {
    "methods": ["local", "oauth"],
    "providers": ["google", "facebook"]
  }
}
```

### Step 2: Create User Profiles Collection

Create a collection to store user type information and metadata:

```javascript
// Create user_profiles collection
POST /api/projects/{projectId}/schemas
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "user_profiles",
  "fields": [
    {
      "name": "userId",
      "type": "string",
      "required": true,
      "indexed": true,
      "unique": true
    },
    {
      "name": "userType",
      "type": "enum",
      "required": true,
      "indexed": true,
      "validation": {
        "enum": ["customer", "vendor", "rider", "admin"]
      }
    },
    {
      "name": "vendorId",
      "type": "string",
      "indexed": true
    },
    {
      "name": "status",
      "type": "string",
      "default": "active",
      "validation": {
        "enum": ["active", "inactive", "suspended", "pending"]
      }
    },
    {
      "name": "metadata",
      "type": "object"
    },
    {
      "name": "phone",
      "type": "string"
    },
    {
      "name": "address",
      "type": "object"
    }
  ],
  "permissions": [
    {
      "role": "customer",
      "actions": ["read", "update"],
      "conditions": {
        "userId": "$userId"
      }
    },
    {
      "role": "vendor",
      "actions": ["read", "update"],
      "conditions": {
        "userId": "$userId"
      }
    },
    {
      "role": "rider",
      "actions": ["read", "update"],
      "conditions": {
        "userId": "$userId"
      }
    },
    {
      "role": "admin",
      "actions": ["create", "read", "update", "delete"]
    },
    {
      "role": "authenticated",
      "actions": ["read"]
    }
  ],
  "indexes": [
    {
      "fields": { "userId": 1 },
      "options": { "unique": true }
    },
    {
      "fields": { "userType": 1, "status": 1 }
    }
  ]
}
```

### Step 3: Create Vendors Collection

```javascript
// Create vendors collection
POST /api/projects/{projectId}/schemas
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "vendors",
  "fields": [
    {
      "name": "ownerId",
      "type": "string",
      "required": true,
      "indexed": true
    },
    {
      "name": "businessName",
      "type": "string",
      "required": true
    },
    {
      "name": "slug",
      "type": "string",
      "required": true,
      "unique": true,
      "indexed": true
    },
    {
      "name": "description",
      "type": "string"
    },
    {
      "name": "status",
      "type": "string",
      "default": "pending",
      "validation": {
        "enum": ["pending", "active", "suspended", "rejected"]
      },
      "indexed": true
    },
    {
      "name": "rating",
      "type": "number",
      "default": 0
    },
    {
      "name": "totalOrders",
      "type": "number",
      "default": 0
    },
    {
      "name": "address",
      "type": "object"
    },
    {
      "name": "contactInfo",
      "type": "object"
    },
    {
      "name": "businessHours",
      "type": "object"
    }
  ],
  "permissions": [
    {
      "role": "vendor",
      "actions": ["read", "update"],
      "conditions": {
        "ownerId": "$userId"
      }
    },
    {
      "role": "customer",
      "actions": ["read"]
    },
    {
      "role": "admin",
      "actions": ["create", "read", "update", "delete"]
    },
    {
      "role": "authenticated",
      "actions": ["read"]
    }
  ]
}
```

### Step 4: Create Products Collection

```javascript
// Create products collection
POST /api/projects/{projectId}/schemas
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "products",
  "fields": [
    {
      "name": "vendorId",
      "type": "string",
      "required": true,
      "indexed": true
    },
    {
      "name": "name",
      "type": "string",
      "required": true
    },
    {
      "name": "description",
      "type": "string"
    },
    {
      "name": "price",
      "type": "number",
      "required": true
    },
    {
      "name": "category",
      "type": "string",
      "indexed": true
    },
    {
      "name": "imageUrl",
      "type": "string"
    },
    {
      "name": "status",
      "type": "string",
      "default": "active",
      "validation": {
        "enum": ["active", "inactive", "out_of_stock"]
      },
      "indexed": true
    },
    {
      "name": "stock",
      "type": "number",
      "default": 0
    }
  ],
  "permissions": [
    {
      "role": "vendor",
      "actions": ["create", "read", "update", "delete"],
      "conditions": {
        "vendorId": "$user.metadata.vendorId"
      }
    },
    {
      "role": "customer",
      "actions": ["read"]
    },
    {
      "role": "admin",
      "actions": ["create", "read", "update", "delete"]
    },
    {
      "role": "authenticated",
      "actions": ["read"]
    }
  ],
  "indexes": [
    {
      "fields": { "vendorId": 1, "status": 1 }
    },
    {
      "fields": { "category": 1, "status": 1 }
    }
  ]
}
```

### Step 5: Create Orders Collection

```javascript
// Create orders collection
POST /api/projects/{projectId}/schemas
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "orders",
  "fields": [
    {
      "name": "customerId",
      "type": "string",
      "required": true,
      "indexed": true
    },
    {
      "name": "vendorId",
      "type": "string",
      "required": true,
      "indexed": true
    },
    {
      "name": "riderId",
      "type": "string",
      "indexed": true
    },
    {
      "name": "items",
      "type": "array",
      "required": true
    },
    {
      "name": "subtotal",
      "type": "number",
      "required": true
    },
    {
      "name": "deliveryFee",
      "type": "number",
      "default": 0
    },
    {
      "name": "total",
      "type": "number",
      "required": true
    },
    {
      "name": "status",
      "type": "string",
      "default": "pending",
      "validation": {
        "enum": ["pending", "confirmed", "preparing", "ready", "assigned", "picked_up", "on_the_way", "delivered", "cancelled"]
      },
      "indexed": true
    },
    {
      "name": "deliveryAddress",
      "type": "object",
      "required": true
    },
    {
      "name": "paymentStatus",
      "type": "string",
      "default": "pending",
      "validation": {
        "enum": ["pending", "paid", "failed", "refunded"]
      }
    },
    {
      "name": "trackingNumber",
      "type": "string",
      "unique": true
    }
  ],
  "permissions": [
    {
      "role": "customer",
      "actions": ["create", "read", "update"],
      "conditions": {
        "customerId": "$userId"
      }
    },
    {
      "role": "vendor",
      "actions": ["read", "update"],
      "conditions": {
        "vendorId": "$user.metadata.vendorId"
      }
    },
    {
      "role": "rider",
      "actions": ["read", "update"],
      "conditions": {
        "riderId": "$userId"
      }
    },
    {
      "role": "admin",
      "actions": ["create", "read", "update", "delete"]
    }
  ],
  "indexes": [
    {
      "fields": { "customerId": 1, "status": 1 }
    },
    {
      "fields": { "vendorId": 1, "status": 1 }
    },
    {
      "fields": { "riderId": 1, "status": 1 }
    },
    {
      "fields": { "status": 1, "createdAt": -1 }
    }
  ],
  "settings": {
    "enableRealtime": true,
    "enableAudit": true
  }
}
```

### Step 6: Initialize Multi-Role Feature

**NEW**: Use the ready-to-use Multi-Role Feature for simplified role management!

```javascript
// Step 1: Initialize multi-role feature (auto-initializes with default templates)
GET /api/projects/{projectId}/multi-role
Authorization: Bearer {token}

// Response includes pre-built roles: owner, super_admin, admin, support, seller, vendor, rider, customer
// All roles are ready to use with default permissions
```

### Step 7: User Registration Flow (Using Multi-Role Feature)

#### Customer Registration

```javascript
// ✅ NEW: Role-based signup - Role is in the URL path!
POST /api/auth/local/signup/customer
Content-Type: application/json

{
  "email": "customer@example.com",
  "password": "SecurePass123!",
  "firstName": "John",
  "lastName": "Doe",
  "projectId": "507f1f77bcf86cd799439050"
}

// Response - Role automatically assigned!
{
  "message": "Registration successful. Please check your email for verification.",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "507f1f77bcf86cd799439060",
    "email": "customer@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "customRole": "customer"  // ✅ Automatically assigned
  },
  "role": {
    "slug": "customer",
    "name": "Customer",
    "description": "Standard customer access"
  }
}

// Step 2: Create user profile (optional - for additional metadata)
POST /api/projects/{projectId}/collections/{userProfilesCollectionId}/data
Authorization: Bearer {token}
Content-Type: application/json

{
  "userId": "507f1f77bcf86cd799439060",
  "userType": "customer",
  "status": "active",
  "metadata": {
    "preferredPaymentMethod": "card",
    "favoriteCategories": ["pizza", "burgers"]
  },
  "phone": "+1234567890",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "state": "NY",
    "zipCode": "10001"
  }
}
```

#### Vendor Registration

```javascript
// ✅ NEW: Signup directly as vendor - Role in URL!
POST /api/auth/local/signup/vendor
Content-Type: application/json

{
  "email": "vendor@restaurant.com",
  "password": "SecurePass123!",
  "firstName": "Jane",
  "lastName": "Smith",
  "projectId": "507f1f77bcf86cd799439050"
}

// Response - Vendor role automatically assigned!
{
  "message": "Registration successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "{vendor_user_id}",
    "email": "vendor@restaurant.com",
    "customRole": "vendor"  // ✅ Automatically assigned
  },
  "role": {
    "slug": "vendor",
    "name": "Vendor",
    "description": "Vendor access to manage business operations"
  }
}

// Step 2: Create user profile
POST /api/projects/{projectId}/collections/{userProfilesCollectionId}/data
Authorization: Bearer {token}
Content-Type: application/json

{
  "userId": "{vendor_user_id}",
  "userType": "vendor",
  "status": "pending",
  "metadata": {
    "businessLicense": "BL123456",
    "taxId": "TAX789012"
  }
}

// Step 3: Create vendor record (admin approval required)
POST /api/projects/{projectId}/collections/{vendorsCollectionId}/data
Authorization: Bearer {admin_token}
Content-Type: application/json

{
  "ownerId": "{vendor_user_id}",
  "businessName": "Pizza Palace",
  "slug": "pizza-palace",
  "description": "Best pizza in town!",
  "status": "pending",
  "address": {
    "street": "456 Food Ave",
    "city": "New York",
    "state": "NY",
    "zipCode": "10002"
  },
  "contactInfo": {
    "phone": "+1234567891",
    "email": "vendor@restaurant.com"
  },
  "businessHours": {
    "monday": { "open": "10:00", "close": "22:00" },
    "tuesday": { "open": "10:00", "close": "22:00" },
    "wednesday": { "open": "10:00", "close": "22:00" },
    "thursday": { "open": "10:00", "close": "22:00" },
    "friday": { "open": "10:00", "close": "23:00" },
    "saturday": { "open": "11:00", "close": "23:00" },
    "sunday": { "open": "12:00", "close": "21:00" }
  }
}

// Step 4: Update user profile with vendorId
PATCH /api/projects/{projectId}/collections/{userProfilesCollectionId}/data/{profile_id}
Authorization: Bearer {admin_token}
Content-Type: application/json

{
  "vendorId": "{vendor_id}",
  "status": "active"
}
```

#### Rider Registration

```javascript
// ✅ NEW: Signup directly as rider - Role in URL!
POST /api/auth/local/signup/rider
Content-Type: application/json

{
  "email": "rider@delivery.com",
  "password": "SecurePass123!",
  "firstName": "Mike",
  "lastName": "Johnson",
  "projectId": "507f1f77bcf86cd799439050"
}

// Response - Rider role automatically assigned!
{
  "message": "Registration successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "{rider_user_id}",
    "email": "rider@delivery.com",
    "customRole": "rider"  // ✅ Automatically assigned
  },
  "role": {
    "slug": "rider",
    "name": "Rider",
    "description": "Delivery rider access"
  }
}

// Step 2: Create user profile
POST /api/projects/{projectId}/collections/{userProfilesCollectionId}/data
Authorization: Bearer {token}
Content-Type: application/json

{
  "userId": "{rider_user_id}",
  "userType": "rider",
  "status": "pending",
  "metadata": {
    "vehicleType": "motorcycle",
    "licenseNumber": "DL123456",
    "availability": true
  },
  "phone": "+1234567892"
}
```

#### OAuth Signup with Role

```javascript
// ✅ NEW: OAuth signup with role in URL!
GET /api/auth/oauth/signup/vendor/google/{projectId}

// Redirects to Google OAuth, then assigns vendor role automatically
// User completes OAuth flow and gets vendor role assigned
```

### Step 8: Configure Multi-Role Feature (Optional)

Customize roles and permissions from the dashboard:

```javascript
// Get current configuration
GET /api/projects/{projectId}/multi-role
Authorization: Bearer {token}

// Toggle roles on/off
PATCH /api/projects/{projectId}/multi-role/roles/vendor/toggle
Authorization: Bearer {token}
Content-Type: application/json
{
  "isEnabled": true
}

// Update role settings
PATCH /api/projects/{projectId}/multi-role/roles/vendor
Authorization: Bearer {token}
Content-Type: application/json
{
  "requiresApproval": false,
  "requiresPayment": true,
  "requiresKYC": true
}

// Configure collection permissions for vendor role
PATCH /api/projects/{projectId}/multi-role/roles/vendor/collections/{productsCollectionId}/permissions
Authorization: Bearer {token}
Content-Type: application/json
{
  "actions": ["create", "read", "update", "delete"],
  "conditions": {
    "vendorId": "$user.metadata.vendorId"
  }
}
```

### Step 9: Complete Order Flow

#### Customer Places Order

```javascript
// Step 1: Customer browses products
GET /api/projects/{projectId}/collections/{productsCollectionId}/data?filter={"status":"active","vendorId":"{vendor_id}"}
Authorization: Bearer {customer_token}

// Step 2: Customer creates order
POST /api/projects/{projectId}/collections/{ordersCollectionId}/data
Authorization: Bearer {customer_token}
Content-Type: application/json

{
  "customerId": "{customer_user_id}",
  "vendorId": "{vendor_id}",
  "items": [
    {
      "productId": "{product_id}",
      "name": "Margherita Pizza",
      "quantity": 2,
      "price": 15.99
    },
    {
      "productId": "{product_id_2}",
      "name": "Coca Cola",
      "quantity": 1,
      "price": 2.99
    }
  ],
  "subtotal": 34.97,
  "deliveryFee": 5.00,
  "total": 39.97,
  "deliveryAddress": {
    "street": "123 Main St",
    "city": "New York",
    "state": "NY",
    "zipCode": "10001",
    "instructions": "Ring doorbell twice"
  },
  "status": "pending",
  "paymentStatus": "pending"
}

// Response
{
  "message": "Data created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439070",
    "customerId": "{customer_user_id}",
    "vendorId": "{vendor_id}",
    "items": [...],
    "total": 39.97,
    "status": "pending",
    "trackingNumber": "ORD-2024-001234",
    "createdAt": "2024-01-15T10:30:00Z"
  }
}
```

#### Vendor Confirms Order

```javascript
// Vendor receives real-time notification via Socket.IO
// Socket event: order:new

// Vendor confirms order
PATCH /api/projects/{projectId}/collections/{ordersCollectionId}/data/{order_id}
Authorization: Bearer {vendor_token}
Content-Type: application/json

{
  "status": "confirmed"
}

// Vendor updates to preparing
PATCH /api/projects/{projectId}/collections/{ordersCollectionId}/data/{order_id}
Authorization: Bearer {vendor_token}
Content-Type: application/json

{
  "status": "preparing"
}

// Vendor marks as ready
PATCH /api/projects/{projectId}/collections/{ordersCollectionId}/data/{order_id}
Authorization: Bearer {vendor_token}
Content-Type: application/json

{
  "status": "ready"
}
```

#### Rider Accepts and Delivers Order

```javascript
// Step 1: Rider views available orders
GET /api/projects/{projectId}/collections/{ordersCollectionId}/data?filter={"status":"ready","riderId":null}
Authorization: Bearer {rider_token}

// Step 2: Rider accepts order
PATCH /api/projects/{projectId}/collections/{ordersCollectionId}/data/{order_id}
Authorization: Bearer {rider_token}
Content-Type: application/json

{
  "riderId": "{rider_user_id}",
  "status": "assigned"
}

// Step 3: Rider picks up order
PATCH /api/projects/{projectId}/collections/{ordersCollectionId}/data/{order_id}
Authorization: Bearer {rider_token}
Content-Type: application/json

{
  "status": "picked_up"
}

// Step 4: Rider updates to on the way
PATCH /api/projects/{projectId}/collections/{ordersCollectionId}/data/{order_id}
Authorization: Bearer {rider_token}
Content-Type: application/json

{
  "status": "on_the_way"
}

// Step 5: Rider marks as delivered
PATCH /api/projects/{projectId}/collections/{ordersCollectionId}/data/{order_id}
Authorization: Bearer {rider_token}
Content-Type: application/json

{
  "status": "delivered"
}
```

### Step 10: Real-Time Order Tracking

```javascript
// Customer subscribes to order updates via Socket.IO
const socket = io('https://mudbase-server.onrender.com', {
  auth: {
    token: customerToken
  }
})

// Join project room
socket.emit('join:project', { projectId: '{projectId}' })

// Subscribe to order updates
socket.on('database:order:update', (data) => {
  console.log('Order status updated:', data)
  // Update UI with new status
  updateOrderStatus(data.status)
})

// Vendor subscribes to new orders
socket.on('database:order:create', (data) => {
  if (data.vendorId === currentVendorId) {
    showNewOrderNotification(data)
  }
})

// Rider subscribes to available orders
socket.on('database:order:update', (data) => {
  if (data.status === 'ready' && !data.riderId) {
    showAvailableOrder(data)
  }
})
```

### Step 11: Admin Dashboard Operations

```javascript
// Admin views all orders
GET /api/projects/{projectId}/collections/{ordersCollectionId}/data?filter={}&limit=100&sort=-createdAt
Authorization: Bearer {admin_token}

// Admin views all vendors
GET /api/projects/{projectId}/collections/{vendorsCollectionId}/data
Authorization: Bearer {admin_token}

// Admin approves vendor
PATCH /api/projects/{projectId}/collections/{vendorsCollectionId}/data/{vendor_id}
Authorization: Bearer {admin_token}
Content-Type: application/json

{
  "status": "active"
}

// Admin suspends vendor
PATCH /api/projects/{projectId}/collections/{vendorsCollectionId}/data/{vendor_id}
Authorization: Bearer {admin_token}
Content-Type: application/json

{
  "status": "suspended"
}

// Admin views analytics
GET /api/projects/{projectId}/usage
Authorization: Bearer {admin_token}
```

### Key Features Demonstrated

1. **Multi-Role Feature**: Ready-to-use role system with pre-built templates
2. **Role-Based Signup**: Role specified in URL path (e.g., `/signup/customer`, `/signup/vendor`)
3. **Project-Based Authentication**: All users authenticate under the same project
4. **Custom User Types**: Using `customRole` field and `user_profiles` collection
5. **Collection-Level Permissions**: Each collection defines who can perform what actions
6. **Automatic Data Filtering**: Users only see/modify data they're allowed to access
7. **Real-Time Updates**: Socket.IO for live order tracking
8. **Role-Based Access**: Vendors see only their orders, customers see only their orders, etc.
9. **Dashboard Configuration**: All role settings managed from project dashboard

### Security Features

1. **Automatic Filtering**: Collection permissions automatically filter queries
2. **Ownership Validation**: Users can only update/delete their own records
3. **Role-Based Permissions**: Different actions allowed based on user type
4. **Audit Logging**: All data operations are logged
5. **Real-Time Events**: Secure Socket.IO connections with authentication

### Benefits of This Approach

- ✅ **Ready-to-Use**: Pre-built role templates, no coding required
- ✅ **Role-Based Signup**: Simple endpoints like `/signup/customer`, `/signup/vendor`
- ✅ **Single Project**: All users in one project, easier management
- ✅ **Flexible Schema**: Easy to add new fields or collections
- ✅ **Automatic Security**: Permissions enforced at the API level
- ✅ **Scalable**: Can handle thousands of vendors, customers, and riders
- ✅ **Real-Time**: Live updates for all stakeholders
- ✅ **Type-Safe**: Schema validation ensures data integrity
- ✅ **Dashboard Configuration**: All settings managed from web interface
- ✅ **All Auth Methods**: Works with Local, OAuth, Magic Link, OTP, Anonymous

