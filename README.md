# Middleware Implementation for Logging and Bearer Token Authentication

## Objective

Learn how to build and integrate middleware functions in an Express.js application to handle request logging and protect routes using a Bearer token. This task helps you understand middleware flow, request validation, and how to enforce secure access to specific endpoints in a Node.js backend.

## Task Description

Create an Express.js server and implement two custom middleware functions. The first middleware should log the HTTP method, request URL, and timestamp for every incoming request. The second middleware should check for an Authorization header that includes a Bearer token. Only requests that include the token `mysecrettoken` should be allowed to access the protected route; all other requests should be denied with appropriate error messages. Apply the logging middleware globally to all routes. Create at least two routes: one public route accessible without authentication, and one protected route that requires the correct Bearer token (`mysecrettoken`) to access. Test both routes using curl or Postman to demonstrate how logging and token-based authentication work together.

## Features

- **Global Request Logging Middleware**: Logs HTTP method, request URL, and timestamp for every incoming request
- **Bearer Token Authentication Middleware**: Validates Authorization header with Bearer token
- **Public Route**: Accessible without authentication
- **Protected Route**: Requires valid Bearer token (`mysecrettoken`)
- **Error Handling**: Returns appropriate error messages for unauthorized requests

## Setup Instructions

### Prerequisites

- Node.js (v14 or higher)
- npm or yarn

### Installation

1. Clone the repository:
```bash
git clone https://github.com/SachinKumarGupta04/express-middleware-logging-auth.git
cd express-middleware-logging-auth
```

2. Install dependencies:
```bash
npm install
```

3. Start the server:
```bash
node server.js
```

The server will start on `http://localhost:3000`

## Implementation

### Server Setup (server.js)

```javascript
const express = require('express');
const app = express();
const PORT = 3000;

// Middleware 1: Request Logging Middleware (Global)
const requestLogger = (req, res, next) => {
  const timestamp = new Date().toISOString();
  console.log(`[${timestamp}] ${req.method} ${req.url}`);
  next();
};

// Middleware 2: Bearer Token Authentication Middleware
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  
  if (!authHeader) {
    return res.status(401).json({ 
      error: 'Authorization header missing',
      message: 'Please provide an Authorization header with Bearer token' 
    });
  }
  
  const token = authHeader.split(' ')[1]; // Extract token from "Bearer <token>"
  
  if (!token) {
    return res.status(401).json({ 
      error: 'Token missing',
      message: 'Authorization header must include a Bearer token' 
    });
  }
  
  if (token !== 'mysecrettoken') {
    return res.status(403).json({ 
      error: 'Invalid token',
      message: 'The provided token is not valid' 
    });
  }
  
  // Token is valid, proceed to the route handler
  next();
};

// Apply logging middleware globally to all routes
app.use(requestLogger);

// Public Route - No authentication required
app.get('/public', (req, res) => {
  res.json({ 
    message: 'Welcome to the public route!',
    description: 'This route is accessible without authentication' 
  });
});

// Protected Route - Requires Bearer token authentication
app.get('/protected', authenticateToken, (req, res) => {
  res.json({ 
    message: 'Welcome to the protected route!',
    description: 'You have successfully authenticated with the Bearer token',
    data: { secretInfo: 'This is sensitive information only for authenticated users' }
  });
});

// Root route
app.get('/', (req, res) => {
  res.json({
    message: 'Express Middleware Demo Server',
    endpoints: {
      public: '/public - No authentication required',
      protected: '/protected - Requires Bearer token: mysecrettoken'
    }
  });
});

// Start server
app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
  console.log('\nAvailable routes:');
  console.log('  GET / - Server information');
  console.log('  GET /public - Public route (no auth required)');
  console.log('  GET /protected - Protected route (requires Bearer token)');
});
```

## Testing Instructions

### Using curl

#### 1. Test the root route:
```bash
curl http://localhost:3000/
```

**Expected Output:**
```json
{
  "message": "Express Middleware Demo Server",
  "endpoints": {
    "public": "/public - No authentication required",
    "protected": "/protected - Requires Bearer token: mysecrettoken"
  }
}
```

#### 2. Test the public route (no authentication):
```bash
curl http://localhost:3000/public
```

**Expected Output:**
```json
{
  "message": "Welcome to the public route!",
  "description": "This route is accessible without authentication"
}
```

#### 3. Test the protected route without token (should fail):
```bash
curl http://localhost:3000/protected
```

**Expected Output:**
```json
{
  "error": "Authorization header missing",
  "message": "Please provide an Authorization header with Bearer token"
}
```

#### 4. Test the protected route with invalid token (should fail):
```bash
curl -H "Authorization: Bearer wrongtoken" http://localhost:3000/protected
```

**Expected Output:**
```json
{
  "error": "Invalid token",
  "message": "The provided token is not valid"
}
```

#### 5. Test the protected route with valid token (should succeed):
```bash
curl -H "Authorization: Bearer mysecrettoken" http://localhost:3000/protected
```

**Expected Output:**
```json
{
  "message": "Welcome to the protected route!",
  "description": "You have successfully authenticated with the Bearer token",
  "data": {
    "secretInfo": "This is sensitive information only for authenticated users"
  }
}
```

### Using Postman

#### Test Public Route:
1. Create a new GET request
2. URL: `http://localhost:3000/public`
3. Click "Send"
4. You should receive a successful response without any authentication

#### Test Protected Route (Unauthorized):
1. Create a new GET request
2. URL: `http://localhost:3000/protected`
3. Click "Send" without adding any headers
4. You should receive a 401 Unauthorized error

#### Test Protected Route (Authorized):
1. Create a new GET request
2. URL: `http://localhost:3000/protected`
3. Go to the "Authorization" tab
4. Select "Bearer Token" from the Type dropdown
5. Enter `mysecrettoken` in the Token field
6. Click "Send"
7. You should receive a successful response with protected data

**Alternative method:**
1. Go to the "Headers" tab
2. Add a new header:
   - Key: `Authorization`
   - Value: `Bearer mysecrettoken`
3. Click "Send"

## Console Output Example

When the server is running and receiving requests, you'll see logging output like:

```
Server is running on http://localhost:3000

Available routes:
  GET / - Server information
  GET /public - Public route (no auth required)
  GET /protected - Protected route (requires Bearer token)

[2025-10-30T08:27:14.523Z] GET /
[2025-10-30T08:27:20.145Z] GET /public
[2025-10-30T08:27:25.789Z] GET /protected
[2025-10-30T08:27:32.456Z] GET /protected
```

Each request is logged with:
- Timestamp (ISO 8601 format)
- HTTP Method (GET, POST, etc.)
- Request URL

## Middleware Flow Explanation

### Request Logging Middleware

```javascript
const requestLogger = (req, res, next) => {
  const timestamp = new Date().toISOString();
  console.log(`[${timestamp}] ${req.method} ${req.url}`);
  next(); // Pass control to the next middleware
};
```

- Applied globally using `app.use(requestLogger)`
- Executes for every incoming request
- Logs request details before passing control to the next middleware
- Calls `next()` to continue to the next middleware or route handler

### Bearer Token Authentication Middleware

```javascript
const authenticateToken = (req, res, next) => {
  // 1. Check if Authorization header exists
  const authHeader = req.headers['authorization'];
  if (!authHeader) {
    return res.status(401).json({ error: '...' });
  }
  
  // 2. Extract token from "Bearer <token>" format
  const token = authHeader.split(' ')[1];
  if (!token) {
    return res.status(401).json({ error: '...' });
  }
  
  // 3. Validate the token
  if (token !== 'mysecrettoken') {
    return res.status(403).json({ error: '...' });
  }
  
  // 4. Token is valid, proceed
  next();
};
```

- Applied selectively to specific routes (e.g., `/protected`)
- Checks for Authorization header presence
- Extracts and validates the Bearer token
- Returns appropriate HTTP status codes:
  - `401 Unauthorized`: Missing or malformed authentication
  - `403 Forbidden`: Invalid token
  - Calls `next()` only if authentication succeeds

## Key Concepts Demonstrated

1. **Global Middleware**: Applied to all routes using `app.use()`
2. **Route-Specific Middleware**: Applied to individual routes as parameters
3. **Middleware Chain**: Multiple middleware functions executed in sequence
4. **Request/Response Cycle**: Understanding when to call `next()` vs returning a response
5. **Authorization Header**: Standard Bearer token format (`Bearer <token>`)
6. **HTTP Status Codes**: Proper use of 401 (Unauthorized) and 403 (Forbidden)
7. **Error Handling**: Providing meaningful error messages to clients

## Package.json

```json
{
  "name": "express-middleware-logging-auth",
  "version": "1.0.0",
  "description": "Demonstration of Express.js middleware for logging and Bearer token authentication",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "keywords": [
    "express",
    "middleware",
    "authentication",
    "bearer-token",
    "logging",
    "nodejs"
  ],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

## Security Notes

⚠️ **Important**: This implementation is for educational purposes only.

In production applications:

1. **Never hardcode tokens**: Use environment variables
2. **Use JWT tokens**: Instead of simple string comparison
3. **Implement token expiration**: Tokens should have limited validity
4. **Use HTTPS**: Always transmit tokens over secure connections
5. **Store tokens securely**: Use secure storage mechanisms on the client
6. **Implement refresh tokens**: For better security and user experience
7. **Add rate limiting**: Prevent brute force attacks
8. **Log security events**: Track authentication failures

## Extensions and Improvements

Potential enhancements to this basic implementation:

- Add more routes with different access levels
- Implement role-based access control (RBAC)
- Add request body parsing middleware
- Implement JWT token generation and validation
- Add error handling middleware
- Include CORS middleware
- Add request rate limiting
- Implement token refresh mechanism
- Add middleware for input validation
- Include helmet.js for security headers

## License

ISC

## Repository

[GitHub Repository](https://github.com/SachinKumarGupta04/express-middleware-logging-auth)
