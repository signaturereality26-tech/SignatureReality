# 19 - Security Framework

## Comprehensive Security Implementation

### Security Layers

```
┌─────────────────────────────────────┐
│   Authentication & Authorization    │
├─────────────────────────────────────┤
│   Data Encryption & Protection      │
├─────────────────────────────────────┤
│   Audit Logging & Compliance        │
├─────────────────────────────────────┤
│   Access Control & Permissions      │
├─────────────────────────────────────┤
│   Session Management & Protection   │
└─────────────────────────────────────┘
```

## Authentication System

```javascript
function authenticateUser(email, password) {
  try {
    // 1. Check user exists
    const user = getUserByEmail(email);
    if (!user) {
      logSecurityEvent('LOGIN_FAILED_USER_NOT_FOUND', email);
      return { success: false, message: 'Invalid credentials' };
    }
    
    // 2. Check if user is active
    if (user.Status !== 'Active') {
      logSecurityEvent('LOGIN_FAILED_INACTIVE_USER', email);
      return { success: false, message: 'User account is inactive' };
    }
    
    // 3. Check if user is locked
    if (user.IsLocked) {
      const lockTime = calculateLockTime(user.LockedUntil);
      logSecurityEvent('LOGIN_FAILED_LOCKED_USER', email);
      return { 
        success: false, 
        message: `Account locked. Try again after ${lockTime} minutes` 
      };
    }
    
    // 4. Verify password (hashed)
    const passwordHash = hashPassword(password);
    if (user.PasswordHash !== passwordHash) {
      // Increment failed attempts
      user.FailedLoginAttempts = (user.FailedLoginAttempts || 0) + 1;
      
      // Lock after 5 failed attempts
      if (user.FailedLoginAttempts >= 5) {
        user.IsLocked = true;
        user.LockedUntil = new Date(Date.now() + 30 * 60 * 1000); // 30 mins
        logSecurityEvent('USER_LOCKED', email);
      }
      
      updateUser(user);
      logSecurityEvent('LOGIN_FAILED_WRONG_PASSWORD', email);
      return { success: false, message: 'Invalid credentials' };
    }
    
    // 5. Reset failed attempts
    user.FailedLoginAttempts = 0;
    user.LastLoginDate = new Date();
    updateUser(user);
    
    // 6. Create session
    const session = {
      SessionID: generateUniqueId('SESSION'),
      UserID: user.UserID,
      Email: email,
      Role: user.Role,
      Permissions: getPermissions(user.Role),
      LoginTime: new Date(),
      ExpiryTime: new Date(Date.now() + CONFIG.SESSION_TIMEOUT),
      IPAddress: getClientIP(),
      UserAgent: getClientUserAgent(),
      Active: true
    };
    
    storeSession(session);
    logSecurityEvent('LOGIN_SUCCESS', email);
    
    return { 
      success: true, 
      sessionId: session.SessionID,
      message: 'Login successful'
    };
    
  } catch (error) {
    Logger.log('Authentication error: ' + error);
    logSecurityEvent('AUTH_ERROR', email, error.message);
    return { success: false, message: 'Authentication failed' };
  }
}

function hashPassword(password) {
  // Use bcrypt or similar (simplified here)
  return Utilities.computeDigest(
    Utilities.DigestAlgorithm.SHA_256, 
    password
  );
}
```

## Authorization & Role-Based Access Control (RBAC)

```javascript
const Roles = {
  ADMIN: 'Admin',
  MANAGER: 'Manager',
  AGENT: 'Agent',
  GUEST: 'Guest'
};

const PermissionMatrix = {
  Admin: {
    leads: ['read', 'create', 'update', 'delete'],
    properties: ['read', 'create', 'update', 'delete'],
    agents: ['read', 'create', 'update', 'delete'],
    commission: ['read', 'create', 'update', 'delete'],
    reports: ['read', 'create', 'delete'],
    security: ['read', 'update']
  },
  Manager: {
    leads: ['read', 'create', 'update'],
    properties: ['read', 'create', 'update'],
    agents: ['read', 'update'],
    commission: ['read'],
    reports: ['read', 'create'],
    security: ['read']
  },
  Agent: {
    leads: ['read', 'create', 'update'],
    properties: ['read'],
    agents: [],
    commission: ['read'],
    reports: ['read'],
    security: []
  },
  Guest: {
    leads: ['read'],
    properties: ['read'],
    agents: [],
    commission: [],
    reports: [],
    security: []
  }
};

function checkPermission(sessionId, resource, action) {
  try {
    const session = getSession(sessionId);
    
    if (!session || !session.Active) {
      logSecurityEvent('PERMISSION_CHECK_INVALID_SESSION', sessionId);
      return false;
    }
    
    // Check session expiry
    if (new Date() > session.ExpiryTime) {
      invalidateSession(sessionId);
      logSecurityEvent('SESSION_EXPIRED', session.Email);
      return false;
    }
    
    // Check permissions
    const rolePermissions = PermissionMatrix[session.Role];
    const resourcePermissions = rolePermissions[resource] || [];
    
    const hasPermission = resourcePermissions.includes(action);
    
    if (!hasPermission) {
      logSecurityEvent('PERMISSION_DENIED', 
        session.Email, 
        `${resource}.${action}`
      );
    }
    
    return hasPermission;
    
  } catch (error) {
    Logger.log('Permission check error: ' + error);
    return false;
  }
}

function getPermissions(role) {
  return PermissionMatrix[role] || {};
}
```

## Session Management

```javascript
function validateSession(sessionId) {
  try {
    const session = getSession(sessionId);
    
    if (!session) {
      return { valid: false, message: 'Session not found' };
    }
    
    // Check if active
    if (!session.Active) {
      return { valid: false, message: 'Session inactive' };
    }
    
    // Check expiry
    const now = new Date();
    if (now > session.ExpiryTime) {
      invalidateSession(sessionId);
      return { valid: false, message: 'Session expired' };
    }
    
    // Extend session on valid access
    session.LastAccessTime = now;
    session.ExpiryTime = new Date(
      now.getTime() + CONFIG.SESSION_TIMEOUT
    );
    updateSession(session);
    
    return { 
      valid: true, 
      session: session 
    };
    
  } catch (error) {
    Logger.log('Session validation error: ' + error);
    return { valid: false, message: 'Session validation failed' };
  }
}

function invalidateSession(sessionId) {
  const session = getSession(sessionId);
  if (session) {
    session.Active = false;
    session.LogoutTime = new Date();
    updateSession(session);
    logSecurityEvent('LOGOUT', session.Email);
  }
}

function logoutUser(sessionId) {
  invalidateSession(sessionId);
  return { success: true, message: 'Logged out successfully' };
}
```

## Data Encryption

```javascript
function encryptSensitiveData(data, encryptionKey = null) {
  try {
    // Sensitive fields: mobile, email, bank account details
    const sensitiveFields = ['Mobile', 'Email', 'AccountNumber', 'IFSC'];
    const encrypted = { ...data };
    
    sensitiveFields.forEach(field => {
      if (encrypted[field]) {
        encrypted[field] = encryptValue(encrypted[field], encryptionKey);
      }
    });
    
    return encrypted;
    
  } catch (error) {
    Logger.log('Encryption error: ' + error);
    return data;
  }
}

function encryptValue(value, key) {
  // Use Apps Script encryption or service
  const encrypted = Utilities.base64Encode(value);
  return encrypted;
}

function decryptSensitiveData(data, decryptionKey = null) {
  const sensitiveFields = ['Mobile', 'Email', 'AccountNumber', 'IFSC'];
  const decrypted = { ...data };
  
  sensitiveFields.forEach(field => {
    if (decrypted[field] && isEncrypted(decrypted[field])) {
      decrypted[field] = decryptValue(decrypted[field], decryptionKey);
    }
  });
  
  return decrypted;
}

function decryptValue(value, key) {
  const decrypted = Utilities.base64Decode(value);
  return Utilities.newBlob(decrypted).getDataAsString();
}
```

## Audit Logging

```javascript
function logSecurityEvent(eventType, user, action = '', details = '') {
  try {
    const event = {
      EventID: generateUniqueId('EVENT'),
      EventType: eventType, // LOGIN_SUCCESS, PERMISSION_DENIED, etc.
      User: user,
      Action: action,
      Details: details,
      IPAddress: getClientIP(),
      UserAgent: getClientUserAgent(),
      Timestamp: new Date(),
      Status: 'Logged'
    };
    
    storeAuditLog(event);
    
    // Alert on critical events
    if (['PERMISSION_DENIED', 'USER_LOCKED', 'AUTH_ERROR']
        .includes(eventType)) {
      sendSecurityAlert(event);
    }
    
  } catch (error) {
    Logger.log('Error logging security event: ' + error);
  }
}

function logDataAccess(userId, resource, action, recordId) {
  logSecurityEvent(
    'DATA_ACCESS',
    userId,
    `${resource}.${action}`,
    `Record: ${recordId}`
  );
}

function logDataModification(userId, resource, action, recordId, changes) {
  logSecurityEvent(
    'DATA_MODIFICATION',
    userId,
    `${resource}.${action}`,
    `Record: ${recordId}, Changes: ${JSON.stringify(changes)}`
  );
}
```

## Access Control Wrapper

```javascript
function secureFunction(sessionId, resource, action, callback) {
  try {
    // 1. Validate session
    const sessionValidation = validateSession(sessionId);
    if (!sessionValidation.valid) {
      return {
        success: false,
        message: sessionValidation.message
      };
    }
    
    // 2. Check permissions
    const hasPermission = checkPermission(sessionId, resource, action);
    if (!hasPermission) {
      return {
        success: false,
        message: 'Access denied. Insufficient permissions.'
      };
    }
    
    // 3. Log data access
    logDataAccess(
      sessionValidation.session.Email,
      resource,
      action,
      ''
    );
    
    // 4. Execute protected function
    const result = callback();
    
    return {
      success: true,
      data: result
    };
    
  } catch (error) {
    Logger.log('Secure function error: ' + error);
    logSecurityEvent('FUNCTION_ERROR', sessionId, error.message);
    return {
      success: false,
      message: 'Function execution failed'
    };
  }
}
```

---

Next: See [20_ERROR_HANDLING.md](20_ERROR_HANDLING.md) for error handling framework.
