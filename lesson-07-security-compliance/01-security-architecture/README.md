# 7.1 Security Architecture

## Overview

Security architecture in insurance systems implements a defense-in-depth strategy, providing multiple layers of protection for sensitive customer data and business operations. This sub-lesson covers the fundamental security layers and authentication mechanisms.

---

## Defense in Depth

The defense-in-depth approach ensures that if one security layer is compromised, additional layers continue to protect the system.

```
+-----------------------------------------------------------------------------+
|                          SECURITY LAYERS                                     |
+-----------------------------------------------------------------------------+
|                                                                              |
|  +-----------------------------------------------------------------------+  |
|  |                      PERIMETER SECURITY                                |  |
|  |  - WAF (Web Application Firewall)                                      |  |
|  |  - DDoS Protection (CloudFlare, AWS Shield)                           |  |
|  |  - API Gateway Security                                                |  |
|  |  - Network Firewall                                                    |  |
|  +-----------------------------------------------------------------------+  |
|                                    |                                         |
|                                    v                                         |
|  +-----------------------------------------------------------------------+  |
|  |                   AUTHENTICATION & AUTHORIZATION                       |  |
|  |  - OAuth 2.0 / OpenID Connect                                         |  |
|  |  - Multi-Factor Authentication (MFA)                                  |  |
|  |  - Role-Based Access Control (RBAC)                                   |  |
|  |  - API Key Management                                                 |  |
|  |  - JWT Token Validation                                               |  |
|  +-----------------------------------------------------------------------+  |
|                                    |                                         |
|                                    v                                         |
|  +-----------------------------------------------------------------------+  |
|  |                      APPLICATION SECURITY                              |  |
|  |  - Input Validation & Sanitization                                    |  |
|  |  - Output Encoding                                                    |  |
|  |  - SQL Injection Prevention (Parameterized Queries)                   |  |
|  |  - XSS Prevention (Content Security Policy)                           |  |
|  |  - CSRF Protection                                                    |  |
|  |  - Secure Headers                                                     |  |
|  +-----------------------------------------------------------------------+  |
|                                    |                                         |
|                                    v                                         |
|  +-----------------------------------------------------------------------+  |
|  |                        DATA SECURITY                                   |  |
|  |  - Encryption at Rest (AES-256)                                       |  |
|  |  - Encryption in Transit (TLS 1.3)                                    |  |
|  |  - Key Management (HSM / Cloud KMS)                                   |  |
|  |  - Data Masking                                                       |  |
|  |  - Tokenization (PCI-DSS)                                             |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
+-----------------------------------------------------------------------------+
```

### Security Layer Details

| Layer | Components | Purpose |
|-------|------------|---------|
| **Perimeter** | WAF, DDoS Protection, Firewall | Block malicious traffic before it reaches the application |
| **Authentication** | OAuth 2.0, MFA, JWT | Verify user identity |
| **Authorization** | RBAC, API Keys | Control access to resources |
| **Application** | Input Validation, CSP, CSRF | Prevent application-level attacks |
| **Data** | Encryption, Masking, Tokenization | Protect data at rest and in transit |

---

## Authentication Flow

The following diagram illustrates the complete authentication flow in an insurance system:

```
+------------+      +------------+      +------------+      +------------+
|   Client   |      |    API     |      |    Auth    |      |   User     |
|            |      |  Gateway   |      |  Service   |      | Database   |
+-----+------+      +-----+------+      +-----+------+      +-----+------+
      |                   |                   |                   |
      |  1. Login Request |                   |                   |
      |  (username/pwd)   |                   |                   |
      | ----------------->|                   |                   |
      |                   |  2. Authenticate  |                   |
      |                   | ----------------->|                   |
      |                   |                   |  3. Validate User |
      |                   |                   | ----------------->|
      |                   |                   |                   |
      |                   |                   |  4. User Data     |
      |                   |                   | <-----------------|
      |                   |                   |                   |
      |                   |                   |  5. Check MFA     |
      |                   |                   |  (if enabled)     |
      |                   |                   |                   |
      |                   |  6. JWT + Refresh |                   |
      |                   | <-----------------|                   |
      |  7. Tokens        |                   |                   |
      | <-----------------|                   |                   |
      |                   |                   |                   |
      |  8. API Request   |                   |                   |
      |  + Bearer Token   |                   |                   |
      | ----------------->|                   |                   |
      |                   |  9. Verify JWT    |                   |
      |                   | ----------------->|                   |
      |                   |                   |                   |
      |                   |  10. Token Valid  |                   |
      |                   | <-----------------|                   |
      |                   |                   |                   |
      |  11. Response     |                   |                   |
      | <-----------------|                   |                   |
```

### Authentication Steps Explained

1. **Login Request**: Client sends credentials (username/password)
2. **Authenticate**: API Gateway forwards to Auth Service
3. **Validate User**: Auth Service checks credentials against database
4. **User Data**: Database returns user profile if valid
5. **Check MFA**: If enabled, prompt for second factor
6. **JWT + Refresh**: Auth Service generates tokens
7. **Tokens**: Tokens returned to client
8. **API Request**: Client includes Bearer token in subsequent requests
9. **Verify JWT**: Gateway validates token signature and expiration
10. **Token Valid**: Authorization confirmed
11. **Response**: API returns requested data

---

## JWT Token Structure

```json
{
  "header": {
    "alg": "RS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "user-12345",
    "name": "John Smith",
    "email": "john.smith@insurance.com",
    "roles": ["AGENT", "UNDERWRITER"],
    "permissions": ["quote:create", "policy:read"],
    "iat": 1699900000,
    "exp": 1699903600,
    "iss": "insurance-auth-service"
  },
  "signature": "..."
}
```

### Token Types

| Token Type | Lifetime | Purpose |
|------------|----------|---------|
| **Access Token** | 15-60 minutes | API authentication |
| **Refresh Token** | 7-30 days | Obtain new access tokens |
| **ID Token** | 15-60 minutes | User identity information |

---

## Diagrams

- [Security Layers Diagram](./07.1-security-layers.drawio) - Visual representation of defense in depth architecture

---

## Navigation

[Back to Lesson 07](../README.md) | [Next: Access Control](../02-access-control/README.md)
