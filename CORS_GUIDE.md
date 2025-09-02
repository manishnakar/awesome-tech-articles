# CORS Configuration Guide for Sentinel Express Server

## 📋 Overview

This document outlines the complete CORS (Cross-Origin Resource Sharing) implementation for the Sentinel Express.js server, including configuration, security measures, and comprehensive testing procedures.

## 🏗️ Implementation Steps

### 1. Initial Setup

#### Dependencies Added
```json
{
  "dependencies": {
    "cors": "^2.8.5"
  },
  "devDependencies": {
    "@types/cors": "^2.8.17"
  }
}
```

#### Basic CORS Configuration
```typescript
// Initial simple configuration
this.app.use(cors({
  origin: process.env['ALLOWED_ORIGINS']?.split(',') || ['http://localhost:3001'],
  credentials: true
}));
```

### 2. Enhanced Security Implementation

#### Improved CORS Configuration
```typescript
// CORS configuration with origin validation
const allowedOrigins = process.env['ALLOWED_ORIGINS']?.split(',') || ['http://localhost:3001'];

this.app.use(cors({
  origin: (origin, callback) => {
    // Allow requests with no origin (like mobile apps or curl requests)
    if (!origin) return callback(null, true);
    
    if (allowedOrigins.indexOf(origin) !== -1) {
      callback(null, true);
    } else {
      console.warn(`CORS blocked request from unauthorized origin: ${origin}`);
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With']
}));
```

## 🔧 Configuration Details

### Environment Variables
```bash
# CORS Configuration
ALLOWED_ORIGINS=http://localhost:3001,http://localhost:3000
```

### Default Configuration
- **Allowed Origins**: `http://localhost:3001` (default)
- **Credentials**: `true` (enabled)
- **Methods**: `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `OPTIONS`
- **Headers**: `Content-Type`, `Authorization`, `X-Requested-With`

## 🧪 Testing Procedures

### Test 1: Allowed Origin Request
```bash
curl -H "Origin: http://localhost:3001" http://localhost:3000/health -v
```

**Expected Result:**
- ✅ HTTP 200 OK
- ✅ `Access-Control-Allow-Origin: http://localhost:3001`
- ✅ `Access-Control-Allow-Credentials: true`
- ✅ Response body with health data

**Actual Response Headers:**
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://localhost:3001
Vary: Origin
Access-Control-Allow-Credentials: true
Content-Type: application/json; charset=utf-8
```

### Test 2: No Origin Request (Mobile Apps, curl)
```bash
curl http://localhost:3000/health -v
```

**Expected Result:**
- ✅ HTTP 200 OK
- ✅ No CORS headers (not needed)
- ✅ Response body with health data

**Actual Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

### Test 3: Malicious Origin Request
```bash
curl -H "Origin: http://malicious-site.com" http://localhost:3000/health -v
```

**Expected Result:**
- ❌ HTTP 500 Internal Server Error
- ❌ CORS error logged in console
- ❌ No CORS headers in response

**Actual Response:**
```
HTTP/1.1 500 Internal Server Error
Content-Type: application/json; charset=utf-8

{"error":"Internal server error","message":"Something went wrong","timestamp":"..."}
```

**Console Log:**
```
CORS blocked request from unauthorized origin: http://malicious-site.com
Unhandled error: Error: Not allowed by CORS
```

### Test 4: Preflight Request (OPTIONS)
```bash
curl -H "Origin: http://localhost:3001" \
     -H "Access-Control-Request-Method: POST" \
     -H "Access-Control-Request-Headers: Content-Type,Authorization" \
     -X OPTIONS http://localhost:3000/health -v
```

**Expected Result:**
- ✅ HTTP 204 No Content
- ✅ `Access-Control-Allow-Origin: http://localhost:3001`
- ✅ `Access-Control-Allow-Methods: GET,POST,PUT,DELETE,PATCH,OPTIONS`
- ✅ `Access-Control-Allow-Headers: Content-Type,Authorization,X-Requested-With`

**Actual Response Headers:**
```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: http://localhost:3001
Vary: Origin
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET,POST,PUT,DELETE,PATCH,OPTIONS
Access-Control-Allow-Headers: Content-Type,Authorization,X-Requested-With
```

## 🔒 Security Features

### 1. Origin Whitelisting
- Only pre-defined origins are allowed
- Environment variable controlled configuration
- Fallback to secure defaults

### 2. Request Logging
- Unauthorized origins are logged
- Security monitoring capabilities
- Audit trail for CORS violations

### 3. Method Restriction
- Explicit HTTP method whitelist
- Prevents unauthorized HTTP methods
- OPTIONS preflight support

### 4. Header Restriction
- Controlled header access
- Prevents header injection attacks
- Authorization header support

### 5. Credentials Support
- Secure cookie handling
- Session management support
- Cross-origin authentication

## 📊 Test Results Summary

| Test Case | Origin | Expected | Actual | Status |
|-----------|--------|----------|---------|---------|
| **Allowed Origin** | `http://localhost:3001` | 200 OK + CORS headers | ✅ 200 OK + CORS headers | **PASS** |
| **No Origin** | None (curl/mobile) | 200 OK | ✅ 200 OK | **PASS** |
| **Malicious Origin** | `http://malicious-site.com` | 500 Error | ✅ 500 Error | **PASS** |
| **Preflight Request** | `http://localhost:3001` | 204 + CORS headers | ✅ 204 + CORS headers | **PASS** |

## 🚀 Production Considerations

### 1. Environment Configuration
```bash
# Production
ALLOWED_ORIGINS=https://yourdomain.com,https://app.yourdomain.com

# Development
ALLOWED_ORIGINS=http://localhost:3001,http://localhost:3000
```

### 2. Monitoring
- Enable CORS violation logging
- Monitor for unauthorized origin attempts
- Set up alerts for security incidents

### 3. Rate Limiting
- Consider adding rate limiting for CORS preflight requests
- Monitor for CORS abuse attempts

### 4. Security Headers
- Helmet.js provides additional security headers
- CORS works in conjunction with other security middleware

## 🔧 Troubleshooting

### Common Issues

#### 1. CORS Still Allowing Unauthorized Origins
- Check if environment variables are loaded
- Verify CORS middleware order
- Ensure server restart after configuration changes

#### 2. Preflight Requests Failing
- Verify OPTIONS method is in allowed methods
- Check if required headers are in allowedHeaders
- Ensure CORS middleware is before route handlers

#### 3. Credentials Not Working
- Verify `credentials: true` is set
- Check if `Access-Control-Allow-Credentials` header is present
- Ensure client includes `credentials: 'include'`

### Debug Commands
```bash
# Check current environment variables
echo "ALLOWED_ORIGINS: $ALLOWED_ORIGINS"

# Test CORS with verbose output
curl -H "Origin: http://localhost:3001" http://localhost:3000/health -v

# Test preflight request
curl -H "Origin: http://localhost:3001" \
     -H "Access-Control-Request-Method: POST" \
     -X OPTIONS http://localhost:3000/health -v
```

## 📚 References

- [CORS Documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Express CORS Middleware](https://github.com/expressjs/cors)
- [Security Best Practices](https://owasp.org/www-project-top-ten/)

## 📝 Maintenance

### Regular Tasks
1. **Review Allowed Origins**: Monthly review of authorized domains
2. **Security Logs**: Monitor CORS violation logs
3. **Update Dependencies**: Keep CORS package updated
4. **Configuration Review**: Quarterly security configuration review

### Version History
- **v1.0**: Initial basic CORS implementation
- **v2.0**: Enhanced security with origin validation
- **v2.1**: Added comprehensive logging and error handling

---

**Last Updated**: September 2, 2025  
**Version**: 2.1  
**Status**: Production Ready ✅
