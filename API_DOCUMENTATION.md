# 🚀 Lavella Backend API Documentation

A comprehensive guide to the Lavella Backend API with authentication, product management, and category management.

## 📋 Table of Contents

- [Base URL](#base-url)
- [Authentication](#authentication)
- [Headers](#headers)
- [Admin Authentication Routes](#admin-authentication-routes)
- [User Authentication Routes](#user-authentication-routes)
- [Product Routes](#product-routes)
- [Category Routes](#category-routes)
- [Banner Routes](#banner-routes)
- [Error Responses](#error-responses)
- [Image Upload Guidelines](#image-upload-guidelines)

---

## 🌐 Base URL

```
http://localhost:3000/api
```

---

## 🔒 Authentication

The API uses JWT (JSON Web Tokens) for authentication. Tokens are valid for 24 hours and are stored in localStorage for persistent sessions.

### Authentication Types:
- **Public Routes**: No authentication required
- **User Routes**: Requires valid JWT token with user role
- **Admin Routes**: Requires valid JWT token with admin role

### Token Storage & Management:
- Tokens are stored in browser's `localStorage` with key `'authToken'`
- Tokens are automatically validated on page load using `/api/admin/validate` endpoint
- Invalid/expired tokens trigger automatic logout and cleanup
- Frontend authentication persists across page navigations and browser sessions

### Token Format:
```
Authorization: Bearer <your-jwt-token>
```

### Frontend Authentication Flow:
For HTML/CSS/JS frontends using anchor tags and multiple pages:

1. **Initial Authentication:**
   - Login via `/api/admin/login` or `/api/user/login`
   - Store received token in `localStorage.setItem('authToken', token)`
   - Update UI to show authenticated state

2. **Page Navigation with Anchor Tags:**
   ```html
   <!-- Protected pages should validate auth on load -->
   <a href="admin-dashboard.html">Admin Dashboard</a>
   <a href="product-manager.html">Product Manager</a>
   ```

3. **Authentication Check on Each Page:**
   ```javascript
   // Add this to every protected page
   document.addEventListener('DOMContentLoaded', async function() {
       const token = localStorage.getItem('authToken');
       if (!token) {
           window.location.href = 'login.html';
           return;
       }

       // Validate token with server
       const isValid = await validateTokenWithServer(token);
       if (!isValid) {
           localStorage.removeItem('authToken');
           window.location.href = 'login.html';
           return;
       }

       // Token is valid, continue with page functionality
       initializePage();
   });

   async function validateTokenWithServer(token) {
       try {
           const response = await fetch('/api/admin/validate', {
               headers: { 'Authorization': `Bearer ${token}` }
           });
           return response.ok;
       } catch (error) {
           return false;
       }
   }
   ```

4. **Making Authenticated Requests:**
   ```javascript
   async function makeAuthenticatedRequest(url, options = {}) {
       const token = localStorage.getItem('authToken');
       if (!token) {
           window.location.href = 'login.html';
           return;
       }

       const response = await fetch(url, {
           ...options,
           headers: {
               ...options.headers,
               'Authorization': `Bearer ${token}`
           }
       });

       // Handle auth errors
       if (response.status === 401 || response.status === 403) {
           localStorage.removeItem('authToken');
           window.location.href = 'login.html';
           return;
       }

       return response;
   }
   ```

### Token Validation Endpoint:
```http
GET /api/admin/validate
```
**Headers:**
```json
{
  "Authorization": "Bearer <admin-token>"
}
```
**Response (Valid Token):**
```json
{
  "valid": true,
  "user": {
    "id": "admin_id",
    "email": "admin@example.com",
    "role": "admin"
  }
}
```
**Response (Invalid Token):**
```json
{
  "status": "error",
  "message": "Invalid or expired token"
}
```

---

## 📋 Headers

### Standard Headers:
```json
{
  "Content-Type": "application/json",
  "Accept": "application/json"
}
```

### With Authentication:
```json
{
  "Content-Type": "application/json",
  "Accept": "application/json",
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### For File Uploads:
```json
{
  "Content-Type": "multipart/form-data",
  "Accept": "application/json",
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "Accept-Encoding": "gzip"
}
```

**Note**: The API uses image compression middleware, so include `Accept-Encoding: gzip` for optimal performance with file uploads.

---

## 👨‍💼 Admin Authentication Routes

Base path: `/api/admin`

### 1. Create Admin
```http
POST /api/admin/create
```

**Body:**
```json
{
  "name": "Admin Name",
  "email": "admin@example.com",
  "password": "securePassword123"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Admin created successfully",
  "data": {
    "id": "admin_id",
    "name": "Admin Name",
    "email": "admin@example.com",
    "role": "admin"
  }
}
```

### 2. Admin Login
```http
POST /api/admin/login
```

**Body:**
```json
{
  "email": "admin@example.com",
  "password": "securePassword123"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "data": {
    "id": "admin_id",
    "name": "Admin Name",
    "email": "admin@example.com",
    "role": "admin"
  }
}
```

### 3. Admin Logout
```http
POST /api/admin/logout
```

**Headers:**
```json
{
  "Authorization": "Bearer <admin-token>"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Logout successful"
}
```

### 4. Reset Admin Password
```http
POST /api/admin/reset-password
```

**Body:**
```json
{
  "email": "admin@example.com",
  "newPassword": "newSecurePassword123"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Password reset successful"
}
```

### 5. Validate Admin Token
```http
GET /api/admin/validate
```

**Headers:**
```json
{
  "Authorization": "Bearer <admin-token>"
}
```

**Response (Valid Token):**
```json
{
  "valid": true,
  "user": {
    "id": "admin_id",
    "email": "admin@example.com",
    "role": "admin"
  }
}
```

**Response (Invalid Token):**
```json
{
  "status": "error",
  "message": "Invalid or expired token"
}
```

**Usage:**
This endpoint is used to validate stored tokens on page load or before making authenticated requests. It's essential for frontend applications using localStorage for token persistence.

---

## 👤 User Authentication Routes

Base path: `/api/user`

### 1. Create User
```http
POST /api/user/create
```

**Body:**
```json
{
  "name": "User Name",
  "email": "user@example.com",
  "password": "userPassword123"
}
```

**Response:**
```json
{
  "success": true,
  "message": "User created successfully",
  "data": {
    "id": "user_id",
    "name": "User Name",
    "email": "user@example.com",
    "role": "user"
  }
}
```

### 2. User Login
```http
POST /api/user/login
```

**Body:**
```json
{
  "email": "user@example.com",
  "password": "userPassword123"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "data": {
    "id": "user_id",
    "name": "User Name",
    "email": "user@example.com",
    "role": "user"
  }
}
```

### 3. User Logout
```http
POST /api/user/logout
```

**Headers:**
```json
{
  "Authorization": "Bearer <user-token>"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Logout successful"
}
```

### 4. Reset User Password
```http
POST /api/user/reset-password
```

**Body:**
```json
{
  "email": "user@example.com",
  "newPassword": "newUserPassword123"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Password reset successful"
}
```

---

## 🛍️ Product Routes

Base path: `/api/products`

### Public Routes (No Authentication Required)

#### 1. Get All Products
```http
GET /api/products?page=1&limit=10&category=categoryId&search=searchTerm
```

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 10)
- `category` (optional): Filter by category ID
- `search` (optional): Search by product title

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "product_id",
      "title": "Premium Curtain Set",
      "description": "High-quality curtain set with modern design",
      "price": 299.99,
      "discountedPrice": 249.99,
      "isAvailable": true,
      "rating": 4.5,
      "images": [
        {
          "url": "https://s3.amazonaws.com/bucket/image1.jpg",
          "publicId": "products/image1"
        }
      ],
      "category": {
        "id": "category_id",
        "name": "Curtains",
        "description": "Various curtain types"
      },
      "specifications": [
        {
          "id": "spec_id",
          "title": "Material",
          "options": ["Cotton", "Polyester"],
          "product": "product_id"
        }
      ],
      "instructions": [
        {
          "id": "instruction_id",
          "title": "Care Instructions",
          "value": ["Wash in cold water", "Do not bleach"]
        }
      ],
      "faqs": [],
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ],
  "pagination": {
    "currentPage": 1,
    "totalPages": 5,
    "totalProducts": 50
  }
}
```

#### 2. Get Product by ID
```http
GET /api/products/:id
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "product_id",
    "title": "Premium Curtain Set",
    "description": "High-quality curtain set with modern design",
    "price": 299.99,
    "discountedPrice": 249.99,
    "stock": 25,
    "isAvailable": true,
    "rating": 4.5,
    "images": [
      {
        "url": "https://s3.amazonaws.com/bucket/image1.jpg",
        "publicId": "products/image1"
      }
    ],
    "category": {
      "id": "category_id",
      "name": "Curtains",
      "description": "Various curtain types"
    },
    "specifications": [
      {
        "id": "spec_id",
        "title": "Material",
        "options": ["Cotton", "Polyester"],
        "product": "product_id"
      }
    ],
    "instructions": [
      {
        "id": "instruction_id",
        "title": "Care Instructions",
        "value": ["Wash in cold water", "Do not bleach"]
      }
    ],
    "faqs": [],
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

#### 3. Get Products by Category
```http
GET /api/products/category/:categoryId?page=1&limit=10&search=searchTerm
```

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 10)
- `search` (optional): Search by product title

**Response:** Same format as "Get All Products"

### Admin Routes (Admin Authentication Required)

#### 4. Create Product with Specifications and Instructions
```http
POST /api/products
```

**Headers:**
```json
{
  "Content-Type": "multipart/form-data",
  "Authorization": "Bearer <admin-token>",
  "Accept-Encoding": "gzip"
}
```

**Form Data:**
```javascript
// Text fields
category: "60f7b3b3b3b3b3b3b3b3b3b3"
title: "Premium Curtain Set"
description: "High-quality curtain set with modern design"
price: 299.99
discountedPrice: 249.99
isAvailable: true

// JSON fields (send as strings)
specifications: JSON.stringify([
  {
    "title": "Material",
    "options": ["Cotton", "Polyester", "Silk"]
  },
  {
    "title": "Size",
    "options": ["Small", "Medium", "Large", "XL"]
  }
])

instructions: JSON.stringify([
  {
    "title": "Care Instructions",
    "value": ["Wash in cold water", "Do not bleach", "Tumble dry low"]
  },
  {
    "title": "Installation Guide",
    "value": ["Step 1: Measure the window", "Step 2: Install brackets", "Step 3: Hang curtains"]
  }
])

// Files (up to 10 images, max 10MB each)
images: [File1, File2, File3...]
```

**cURL Example:**
```bash
curl -X POST http://localhost:3000/api/products \
  -H "Authorization: Bearer <admin-token>" \
  -H "Accept-Encoding: gzip" \
  -F "category=60f7b3b3b3b3b3b3b3b3b3b3" \
  -F "title=Premium Curtain Set" \
  -F "description=High-quality curtain set" \
  -F "price=299.99" \
  -F "discountedPrice=249.99" \
  -F "isAvailable=true" \
  -F 'specifications=[{"title":"Material","options":["Cotton","Polyester"]}]' \
  -F 'instructions=[{"title":"Care","value":["Wash cold","No bleach"]}]' \
  -F "images=@image1.jpg" \
  -F "images=@image2.jpg"
```

**JavaScript Fetch Example:**
```javascript
const formData = new FormData();
formData.append('category', '60f7b3b3b3b3b3b3b3b3b3b3');
formData.append('title', 'Premium Curtain Set');
formData.append('description', 'High-quality curtain set');
formData.append('price', 299.99);
formData.append('discountedPrice', 249.99);
formData.append('isAvailable', true);

formData.append('specifications', JSON.stringify([
  {
    "title": "Material",
    "options": ["Cotton", "Polyester", "Silk"]
  }
]));

formData.append('instructions', JSON.stringify([
  {
    "title": "Care Instructions",
    "value": ["Wash in cold water", "Do not bleach"]
  }
]));

// Add image files
formData.append('images', imageFile1);
formData.append('images', imageFile2);

fetch('/api/products', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer ' + adminToken,
    'Accept-Encoding': 'gzip'
  },
  body: formData
});
```

**Response:**
```json
{
  "success": true,
  "message": "Product created successfully with specifications and instructions",
  "data": {
    "id": "product_id",
    "title": "Premium Curtain Set",
    "description": "High-quality curtain set with modern design",
    "price": 299.99,
    "discountedPrice": 249.99,
    "stock": 25,
    "isAvailable": true,
    "rating": 0,
    "images": [
      {
        "url": "https://s3.amazonaws.com/bucket/compressed-image1.jpg",
        "publicId": "products/compressed-image1"
      }
    ],
    "category": {
      "id": "category_id",
      "name": "Curtains"
    },
    "specifications": [
      {
        "id": "spec_id",
        "title": "Material",
        "options": ["Cotton", "Polyester", "Silk"],
        "product": "product_id"
      }
    ],
    "instructions": [
      {
        "id": "instruction_id",
        "title": "Care Instructions",
        "value": ["Wash in cold water", "Do not bleach"]
      }
    ],
    "faqs": [],
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

#### 5. Update Product
```http
PUT /api/products/:id
```

**Headers:**
```json
{
  "Content-Type": "multipart/form-data",
  "Authorization": "Bearer <admin-token>",
  "Accept-Encoding": "gzip"
}
```

**Form Data:** Same format as Create Product (all fields optional)

**Response:** Same format as Create Product

#### 6. Delete Product
```http
DELETE /api/products/:id
```

**Headers:**
```json
{
  "Authorization": "Bearer <admin-token>"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Product deleted successfully"
}
```

---

## 📂 Category Routes

Base path: `/api/categories`

### Public Routes (No Authentication Required)

#### 1. Get All Categories
```http
GET /api/categories
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "category_id",
      "name": "Curtains",
      "description": "Various types of curtains",
      "imageUrl": "https://s3.amazonaws.com/bucket/category-image.jpg",
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

#### 2. Get Category by ID
```http
GET /api/categories/:id
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "category_id",
    "name": "Curtains",
    "description": "Various types of curtains",
    "imageUrl": "https://s3.amazonaws.com/bucket/category-image.jpg",
    "isActive": true,
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

### Admin Routes (Admin Authentication Required)

#### 3. Create Category
```http
POST /api/categories
```

**Headers:**
```json
{
  "Content-Type": "application/json",
  "Authorization": "Bearer <admin-token>"
}
```

**Body:**
```json
{
  "name": "Curtains",
  "description": "Various types of curtains",
  "imageUrl": "https://s3.amazonaws.com/bucket/category-image.jpg",
  "isActive": true
}
```

**Response:**
```json
{
  "success": true,
  "message": "Category created successfully",
  "data": {
    "id": "category_id",
    "name": "Curtains",
    "description": "Various types of curtains",
    "imageUrl": "https://s3.amazonaws.com/bucket/category-image.jpg",
    "isActive": true,
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

#### 4. Update Category
```http
PUT /api/categories/:id
```

**Headers:**
```json
{
  "Content-Type": "application/json",
  "Authorization": "Bearer <admin-token>"
}
```

**Body:** Same format as Create Category (all fields optional)

**Response:** Same format as Create Category

#### 5. Delete Category
```http
DELETE /api/categories/:id
```

**Headers:**
```json
{
  "Authorization": "Bearer <admin-token>"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Category deleted successfully"
}
```

---

## 🎯 Banner Routes

Base path: `/api/banners`

### Public Routes (No Authentication Required)

#### 1. Get All Banners
```http
GET /api/banners?page=1&limit=10&category=categoryId&isActive=true
```

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 10)
- `category` (optional): Filter by category ID
- `isActive` (optional): Filter by active status (true/false)

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "banner_id",
      "category": {
        "id": "category_id",
        "title": "Curtains"
      },
      "title": "Summer Sale Banner",
      "subtitle": "Up to 50% Off",
      "description": "Limited time offer on all curtains",
      "image": {
        "url": "https://s3.amazonaws.com/bucket/banners/banner1.jpg",
        "publicId": "banners/banner1"
      },
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ],
  "pagination": {
    "currentPage": 1,
    "totalPages": 3,
    "totalBanners": 25
  }
}
```

#### 2. Get Active Banners
```http
GET /api/banners/active?category=categoryId
```

**Query Parameters:**
- `category` (optional): Filter by category ID

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "banner_id",
      "category": {
        "id": "category_id",
        "title": "Curtains"
      },
      "title": "Summer Sale Banner",
      "subtitle": "Up to 50% Off",
      "description": "Limited time offer on all curtains",
      "image": {
        "url": "https://s3.amazonaws.com/bucket/banners/banner1.jpg",
        "publicId": "banners/banner1"
      },
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

#### 3. Get Banner by ID
```http
GET /api/banners/:id
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "banner_id",
    "category": {
      "id": "category_id",
      "title": "Curtains"
    },
    "title": "Summer Sale Banner",
    "subtitle": "Up to 50% Off",
    "description": "Limited time offer on all curtains",
    "image": {
      "url": "https://s3.amazonaws.com/bucket/banners/banner1.jpg",
      "publicId": "banners/banner1"
    },
    "isActive": true,
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

#### 4. Get Banners by Category
```http
GET /api/banners/category/:categoryId?page=1&limit=10&isActive=true
```

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 10)
- `isActive` (optional): Filter by active status (true/false)

**Response:** Same format as "Get All Banners"

### Admin Routes (Admin Authentication Required)

#### 5. Create Banner
```http
POST /api/banners
```

**Headers:**
```json
{
  "Content-Type": "multipart/form-data",
  "Authorization": "Bearer <admin-token>",
  "Accept-Encoding": "gzip"
}
```

**Form Data:**
```javascript
// Required fields
category: "60f7b3b3b3b3b3b3b3b3b3b3"
subtitle: "Up to 50% Off"
image: File (required - single image file)

// Optional fields
title: "Summer Sale Banner"
description: "Limited time offer on all curtains"
isActive: true
```

**cURL Example:**
```bash
curl -X POST http://localhost:3000/api/banners \
  -H "Authorization: Bearer <admin-token>" \
  -H "Accept-Encoding: gzip" \
  -F "category=60f7b3b3b3b3b3b3b3b3b3b3" \
  -F "title=Summer Sale Banner" \
  -F "subtitle=Up to 50% Off" \
  -F "description=Limited time offer" \
  -F "isActive=true" \
  -F "image=@banner.jpg"
```

**Response:**
```json
{
  "success": true,
  "message": "Banner created successfully",
  "data": {
    "id": "banner_id",
    "category": {
      "id": "category_id",
      "title": "Curtains"
    },
    "title": "Summer Sale Banner",
    "subtitle": "Up to 50% Off",
    "description": "Limited time offer on all curtains",
    "image": {
      "url": "https://s3.amazonaws.com/bucket/banners/compressed-banner1.jpg",
      "publicId": "banners/compressed-banner1"
    },
    "isActive": true,
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

#### 6. Update Banner
```http
PUT /api/banners/:id
```

**Headers:**
```json
{
  "Content-Type": "multipart/form-data",
  "Authorization": "Bearer <admin-token>",
  "Accept-Encoding": "gzip"
}
```

**Form Data:** Same as Create Banner (all fields optional, including image)

**Response:** Same format as Create Banner

#### 7. Delete Banner
```http
DELETE /api/banners/:id
```

**Headers:**
```json
{
  "Authorization": "Bearer <admin-token>"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Banner deleted successfully"
}
```

**Note:** Deleting a banner also removes its associated image from S3 storage.

---

## ❌ Error Responses

### Standard Error Format:
```json
{
  "success": false,
  "message": "Error description",
  "error": "Detailed error message"
}
```

### Common HTTP Status Codes:

#### 400 - Bad Request
```json
{
  "success": false,
  "message": "Invalid request data",
  "error": "Missing required field: title"
}
```

#### 401 - Unauthorized
```json
{
  "success": false,
  "message": "Authentication required",
  "error": "No token provided"
}
```

#### 403 - Forbidden
```json
{
  "success": false,
  "message": "Access denied. Admin role required.",
  "error": "Insufficient permissions"
}
```

#### 404 - Not Found
```json
{
  "success": false,
  "message": "Product not found",
  "error": "No product with the specified ID"
}
```

#### 500 - Internal Server Error
```json
{
  "success": false,
  "message": "Internal server error",
  "error": "Database connection failed"
}
```

---

## 📸 Image Upload Guidelines

### Supported Formats:
- JPEG (.jpg, .jpeg)
- PNG (.png)
- WebP (.webp)
- GIF (.gif)

### Limits:
- **Maximum file size**: 10MB per image
- **Maximum files**: 10 images per request
- **Compression**: Images are automatically compressed to 60% quality using Sharp

### Upload Process:
1. Images are temporarily stored in `/temp` directory
2. Images are compressed automatically
3. Compressed images are uploaded to AWS S3
4. Temporary files are cleaned up
5. S3 URLs are stored in the database

### Headers for Image Upload:
```json
{
  "Content-Type": "multipart/form-data",
  "Authorization": "Bearer <admin-token>",
  "Accept-Encoding": "gzip"
}
```

### Error Handling:
- If any image upload fails, the entire request is rolled back
- All created database entries are removed
- Temporary files are cleaned up automatically

---

## 🔧 Usage Examples

### Complete Product Creation Workflow:

1. **Admin Login:**
```bash
curl -X POST http://localhost:3000/api/admin/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"password"}'
```

2. **Create Category:**
```bash
curl -X POST http://localhost:3000/api/categories \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{"name":"Curtains","description":"Various curtain types"}'
```

3. **Create Product with Images:**
```bash
curl -X POST http://localhost:3000/api/products \
  -H "Authorization: Bearer <token>" \
  -H "Accept-Encoding: gzip" \
  -F "category=<category-id>" \
  -F "title=Premium Curtain" \
  -F "description=High-quality curtain" \
  -F "price=299.99" \
  -F "images=@image1.jpg" \
  -F 'specifications=[{"title":"Material","options":["Cotton"]}]'
```

4. **Get All Products:**
```bash
curl -X GET http://localhost:3000/api/products
```

---

## 🎨 Frontend Implementation Patterns

### Multi-Page Application Structure:
For HTML/CSS/JS applications using anchor tags, organize your frontend as follows:

```
frontend/
├── index.html          # Landing/Login page
├── admin-dashboard.html # Admin dashboard
├── product-manager.html # Product management
├── category-manager.html # Category management
├── css/
│   ├── common.css      # Shared styles
│   └── admin.css       # Admin-specific styles
└── js/
    ├── auth.js         # Authentication utilities
    ├── api.js          # API communication
    ├── common.js       # Shared functionality
    └── admin.js        # Admin-specific logic
```

### Authentication Utilities (auth.js):
```javascript
// Centralized authentication functions
class AuthManager {
    static TOKEN_KEY = 'authToken';
    static LOGIN_PAGE = 'index.html';
    static API_BASE = 'http://localhost:3000/api';

    static getToken() {
        return localStorage.getItem(this.TOKEN_KEY);
    }

    static setToken(token) {
        localStorage.setItem(this.TOKEN_KEY, token);
    }

    static clearToken() {
        localStorage.removeItem(this.TOKEN_KEY);
    }

    static async validateToken() {
        const token = this.getToken();
        if (!token) return false;

        try {
            const response = await fetch(`${this.API_BASE}/admin/validate`, {
                headers: { 'Authorization': `Bearer ${token}` }
            });
            return response.ok;
        } catch (error) {
            return false;
        }
    }

    static async requireAuth() {
        const isValid = await this.validateToken();
        if (!isValid) {
            this.clearToken();
            window.location.href = this.LOGIN_PAGE;
            return false;
        }
        return true;
    }

    static logout() {
        this.clearToken();
        window.location.href = this.LOGIN_PAGE;
    }
}
```

### Protected Page Template:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Admin Dashboard</title>
    <link rel="stylesheet" href="css/common.css">
    <link rel="stylesheet" href="css/admin.css">
</head>
<body>
    <div id="loading">Authenticating...</div>
    <div id="content" style="display: none;">
        <!-- Your page content here -->
        <nav>
            <a href="product-manager.html">Products</a>
            <a href="category-manager.html">Categories</a>
            <button onclick="AuthManager.logout()">Logout</button>
        </nav>
    </div>

    <script src="js/auth.js"></script>
    <script src="js/api.js"></script>
    <script>
        document.addEventListener('DOMContentLoaded', async function() {
            const isAuth = await AuthManager.requireAuth();
            if (isAuth) {
                document.getElementById('loading').style.display = 'none';
                document.getElementById('content').style.display = 'block';
                initializePage();
            }
        });

        function initializePage() {
            // Your page-specific initialization
        }
    </script>
</body>
</html>
```

### API Communication (api.js):
```javascript
class APIClient {
    static BASE_URL = 'http://localhost:3000/api';

    static async request(endpoint, options = {}) {
        const token = AuthManager.getToken();

        const config = {
            ...options,
            headers: {
                ...options.headers,
                ...(token && { 'Authorization': `Bearer ${token}` })
            }
        };

        const response = await fetch(`${this.BASE_URL}${endpoint}`, config);

        // Handle auth errors globally
        if (response.status === 401 || response.status === 403) {
            AuthManager.logout();
            return;
        }

        return response;
    }

    static async get(endpoint) {
        return this.request(endpoint);
    }

    static async post(endpoint, data) {
        return this.request(endpoint, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data)
        });
    }

    static async postFormData(endpoint, formData) {
        return this.request(endpoint, {
            method: 'POST',
            body: formData
        });
    }
}
```

### Usage Examples:

**Login Page (index.html):**
```javascript
async function handleLogin(email, password) {
    try {
        const response = await fetch('/api/admin/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password })
        });

        const data = await response.json();
        if (response.ok) {
            AuthManager.setToken(data.token);
            window.location.href = 'admin-dashboard.html';
        } else {
            showError(data.message);
        }
    } catch (error) {
        showError('Login failed');
    }
}
```

**Product Creation:**
```javascript
async function createProduct(formData) {
    try {
        const response = await APIClient.postFormData('/products', formData);
        const data = await response.json();

        if (response.ok) {
            showSuccess('Product created successfully');
        } else {
            showError(data.message);
        }
    } catch (error) {
        showError('Failed to create product');
    }
}
```

---

## 📝 Notes

- All timestamps are in ISO 8601 format (UTC)
- IDs are MongoDB ObjectIds
- Pagination starts from page 1
- Search is case-insensitive
- JWT tokens expire after 24 hours and are stored in localStorage
- Tokens are validated on each page load using `/api/admin/validate` endpoint
- Stock field has been removed from product model (existing products may still show this field)
- Images are automatically compressed before storage
- The API supports CORS for cross-origin requests
- All routes return JSON responses
- File uploads require `multipart/form-data` content type
- Frontend authentication persists across page navigations using localStorage
- Protected pages should validate authentication before displaying content

---

## 🚀 Quick Start

1. **Setup Environment Variables:**
```env
JWT_SECRET=your-secret-key
MONGODB_URI=your-mongodb-connection-string
AWS_ACCESS_KEY_ID=your-aws-access-key
AWS_SECRET_ACCESS_KEY=your-aws-secret-key
AWS_REGION=your-aws-region
S3_BUCKET_NAME=your-s3-bucket-name
```

2. **Install Dependencies:**
```bash
npm install
```

3. **Start Server:**
```bash
npm start
```

4. **Test API:**
```bash
curl http://localhost:3000/api/products
```

---

*For additional support or questions, please contact the development team.*