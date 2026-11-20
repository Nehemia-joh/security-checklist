# AI Application Security Checklist

A comprehensive security framework for AI application development, incorporating your requirements, OWASP Top 10, and industry best practices.

---

## Table of Contents

1. [Authentication & Access Control](#authentication--access-control)
2. [Data Protection & Encryption](#data-protection--encryption)
3. [Database Security & Injection Prevention](#database-security--injection-prevention)
4. [HTTP Headers & CSRF Protection](#http-headers--csrf-protection)
5. [Dependency & Code Security](#dependency--code-security)
6. [AI-Specific & Advanced Threats](#ai-specific--advanced-threats)
7. [Logging, Monitoring & Incident Response](#logging-monitoring--incident-response)
8. [API Security & Rate Limiting](#api-security--rate-limiting)
9. [OWASP Top 10 Coverage](#owasp-top-10-coverage)

---

## Authentication & Access Control

### [ ] Hide API Keys and Secrets
- **Description**: Store all API keys, credentials, and secrets in environment variables or secure vaults, never in source code.
- **Severity**: CRITICAL
- **OWASP**: 02 - Cryptographic Failures
- **Implementation**:
  - Use `.env` files (gitignored) for local development
  - Use secret management systems (AWS Secrets Manager, HashiCorp Vault, Anthropic Console secrets)
  - Implement key rotation policies
  - Never commit secrets to version control
  - Use different keys for different environments (dev, staging, prod)

### [ ] Use Public Database Keys (Read-Only Where Possible)
- **Description**: Implement role-based access with minimal required permissions. Use read-only keys for public data operations.
- **Severity**: CRITICAL
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Create separate database roles for different operations
  - Grant only SELECT permission for read-only operations
  - Use separate credentials for admin operations
  - Implement principle of least privilege across all services
  - Rotate credentials regularly

### [ ] Restrict Database Permissions
- **Description**: Implement least-privilege database access with granular role-based permissions.
- **Severity**: CRITICAL
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Create separate roles: read-only, read-write, admin
  - Grant SELECT only for read-only role
  - Grant INSERT/UPDATE/DELETE only where necessary
  - Revoke ALTER and DROP permissions from application role
  - Use database users (not superuser) for applications
  - Disable root/admin access from application servers
  - Implement row-level security (RLS) per tenant
  - Audit database permission assignments
  - Test privilege escalation attempts
  - Log all privilege changes
  - Monitor failed access attempts
  - Use separate credentials for different services

### [ ] Enable Row-Level Security (RLS) / Row Security Policies
- **Description**: Enforce database-level access control so users can only see their own data even if they bypass application logic.
- **Severity**: CRITICAL
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Enable RLS at the database level (PostgreSQL, Firebase, etc.)
  - Define policies based on user ID, organization ID, or custom attributes
  - Test that RLS blocks unauthorized access even with direct SQL queries
  - Document all RLS policies
  - Audit RLS permissions during security reviews

### [ ] Enforce Server-Side Authentication
- **Description**: Verify user identity on every API request. Never rely on client-side validation alone.
- **Severity**: CRITICAL
- **OWASP**: 07 - Identification and Authentication Failures
- **Implementation**:
  - Implement token validation on every protected endpoint
  - Use JWT, OAuth2, or session-based authentication
  - Validate token expiration and signature
  - Implement refresh token rotation
  - Never trust Authorization headers alone; validate against database

### [ ] Block Unauthorized Record Access
- **Description**: Validate that users have permission to access requested data. Implement proper authorization checks.
- **Severity**: CRITICAL
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Add ownership checks before returning data (e.g., `user_id` matches request)
  - Implement resource-level authorization
  - Check permissions before delete/update operations
  - Log unauthorized access attempts
  - Use middleware to enforce authorization consistently

### [ ] Secure Session Cookies
- **Description**: Set HttpOnly, Secure, and SameSite attributes on session cookies to prevent theft and CSRF attacks.
- **Severity**: CRITICAL
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  ```
  Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600
  ```
  - HttpOnly: Prevents JavaScript access, mitigates XSS
  - Secure: Only sent over HTTPS
  - SameSite=Strict: Prevents CSRF attacks
  - Max-Age: Set appropriate session duration
  - Path=/: Limit to root path

### [ ] Reset Sessions on Password Change
- **Description**: Invalidate all active sessions when a user changes their password. Force re-authentication.
- **Severity**: CRITICAL
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Clear all active tokens/sessions for the user
  - Revoke refresh tokens
  - Force logout from all devices
  - Send notification of logout to user
  - Log the password change event
  - Send confirmation email of security event
  - Prevent account access until user re-authenticates

### [ ] Lock Accounts After Failed Logins
- **Description**: Temporarily lock account after repeated failed login attempts to prevent brute force attacks.
- **Severity**: CRITICAL
- **OWASP**: 07 - Identification and Authentication Failures
- **Implementation**:
  - Lock after 5 failed attempts within 15 minutes
  - Lock duration: 30 minutes (increases exponentially)
  - Send email notification of lockout
  - Allow unlock via email link or password reset
  - Track lockout in logs
  - Monitor for patterns of account lockouts
  - Implement CAPTCHA before unlock attempts
  - Alert user of suspicious activity

### [ ] Hash Passwords with Strong Algorithms
- **Description**: Use bcrypt, Argon2, or scrypt. Never store plaintext passwords or use weak algorithms like MD5/SHA1.
- **Severity**: CRITICAL
- **OWASP**: 02 - Cryptographic Failures
- **Implementation**:
  - Use bcrypt with cost factor ≥12
  - Use Argon2id for highest security
  - Never use MD5, SHA1, or SHA256 for passwords
  - Always salt passwords
  - Verify password strength before hashing
  - Implement password history to prevent reuse

### [ ] Rate Limit Login Attempts
- **Description**: Prevent brute force attacks by limiting login attempts per IP/user.
- **Severity**: HIGH
- **OWASP**: 07 - Identification and Authentication Failures
- **Implementation**:
  - Limit to 5 attempts per 15 minutes per IP
  - Limit to 10 attempts per hour per username
  - Implement exponential backoff after failed attempts
  - Log all login attempts
  - Lock account temporarily after repeated failures
  - Send alert notifications for suspicious activity

### [ ] Rate Limit Password Reset Requests
- **Description**: Prevent abuse of password reset functionality to enumerate accounts or cause DoS.
- **Severity**: HIGH
- **OWASP**: 07 - Identification and Authentication Failures
- **Implementation**:
  - Limit password reset requests: 3 per hour per email
  - Limit password reset requests: 10 per IP per hour
  - Implement cooldown period between reset requests
  - Log all password reset requests
  - Send notification email of reset request
  - Alert on suspicious reset patterns
  - Implement CAPTCHA after multiple reset attempts
  - Monitor for bulk password reset attacks

### [ ] Add Bot Protection / CAPTCHA
- **Description**: Implement CAPTCHA or similar challenges on login and sensitive endpoints to prevent automated attacks.
- **Severity**: HIGH
- **OWASP**: 07 - Identification and Authentication Failures
- **Implementation**:
  - Use reCAPTCHA v3 or similar service
  - Implement CAPTCHA on login endpoints
  - Implement CAPTCHA on password reset
  - Implement CAPTCHA on API registration endpoints
  - Monitor CAPTCHA failure rates for patterns

### [ ] Expire Password Reset Links
- **Description**: Set short expiration times on password reset tokens and enforce one-time use.
- **Severity**: CRITICAL
- **OWASP**: 07 - Identification and Authentication Failures
- **Implementation**:
  - Expire reset link after 15-30 minutes
  - Make reset links one-time use only
  - Invalidate all other reset tokens when new one is requested
  - Store reset tokens hashed in database
  - Include user ID and expiration in token validation
  - Send reset link via email with clear expiration notice
  - Log all reset link generation and usage
  - Require confirmation when resetting to new password

### [ ] Prevent User Enumeration
- **Description**: Don't reveal whether username/email exists during login or password reset.
- **Severity**: HIGH
- **OWASP**: 07 - Identification and Authentication Failures
- **Implementation**:
  - Return same message for "user not found" and "invalid password"
  - Use timing-safe comparisons to prevent timing attacks
  - Return same HTTP status code (400 or 401) for all authentication failures
  - Don't reveal account existence in password reset
  - Generic message: "If an account exists, you will receive an email"
  - Log enumeration attempts for security monitoring
  - Implement rate limiting on enumeration attempts
  - Monitor for bulk account enumeration attacks

---

## Data Protection & Encryption

### [ ] Encrypt Sensitive Data at Rest
- **Description**: Use AES-256 encryption for sensitive fields in databases (PII, payment data, health info).
- **Severity**: CRITICAL
- **OWASP**: 02 - Cryptographic Failures
- **Implementation**:
  - Identify sensitive fields: SSN, credit cards, health data, emails, phone numbers
  - Use AES-256-GCM encryption
  - Store encryption keys separately from data (use key management service)
  - Encrypt before storing in database
  - Decrypt only when needed
  - Use database encryption (at-rest) in addition to field-level encryption

### [ ] Force HTTPS / TLS Encryption in Transit
- **Description**: Use TLS 1.2+ for all data transmission. Disable HTTP and redirect to HTTPS with HSTS headers.
- **Severity**: CRITICAL
- **OWASP**: 02 - Cryptographic Failures
- **Implementation**:
  - Obtain SSL/TLS certificate from trusted CA
  - Use TLS 1.2 or higher
  - Disable SSLv3, TLS 1.0, TLS 1.1
  - Set HSTS header: `Strict-Transport-Security: max-age=31536000; includeSubDomains`
  - Redirect all HTTP to HTTPS
  - Use strong cipher suites (no export ciphers)
  - Test with SSL Labs for A+ rating

### [ ] Trim API Responses to Necessary Data Only
- **Description**: Only return fields the client needs. Never expose internal IDs, password hashes, or system metadata in responses.
- **Severity**: HIGH
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Use field selection/projection in queries
  - Never include password hashes in API responses
  - Never include raw encryption keys
  - Never include sensitive metadata (timestamps, internal IDs if not needed)
  - Use serialization filters (e.g., Django serializers, GraphQL field selection)
  - Return only required fields in list endpoints

### [ ] Block Unauthorized Field Access/Modification
- **Description**: Prevent users from modifying fields they shouldn't (e.g., 'is_admin', 'payment_status'). Validate all inputs server-side.
- **Severity**: CRITICAL
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Whitelist fields that users can modify
  - Reject requests that attempt to modify protected fields
  - Validate field names against schema
  - Log attempts to modify unauthorized fields
  - Use read-only field annotations in code
  - Separate request DTOs from database models

---

## Database Security & Injection Prevention

### [ ] Parameterize All Database Queries
- **Description**: Use prepared statements and parameterized queries. Never concatenate user input into SQL strings.
- **Severity**: CRITICAL
- **OWASP**: 03 - Injection
- **Implementation**:
  - Use parameterized queries: `SELECT * FROM users WHERE id = ?`
  - Use prepared statements in all languages
  - Use ORMs (Django ORM, SQLAlchemy, Sequelize)
  - Never use string concatenation for SQL: `"SELECT * FROM users WHERE id = " + userId` ❌
  - Escape identifiers separately if needed
  - Test queries with malicious input

### [ ] Validate All User Input on Server-Side
- **Description**: Type check, length check, whitelist allowed values, and reject unexpected input. Never trust client validation.
- **Severity**: CRITICAL
- **OWASP**: 03 - Injection
- **Implementation**:
  - Type validation: Ensure integers are integers, emails are emails
  - Length validation: Enforce min/max string lengths
  - Pattern validation: Use regex for format validation
  - Whitelist validation: Only allow known-good values
  - Range validation: Ensure numbers are within acceptable ranges
  - Use validation libraries (joi, yup, pydantic)
  - Validate on server regardless of client-side validation
  - Return specific error messages for debugging (in development only)

### [ ] Escape User-Generated Content
- **Description**: HTML-escape, URL-encode, or JSON-encode output to prevent XSS attacks based on context.
- **Severity**: CRITICAL
- **OWASP**: 03 - Injection
- **Implementation**:
  - HTML context: `<div>{{ userInput | escape }}</div>` (escapes <, >, &, ", ')
  - URL context: `<a href="{{ url | urlencode }}">` 
  - JavaScript context: `<script>var data = {{ userInput | jsonencode }}</script>`
  - Use templating engine auto-escaping (Jinja2, EJS, Handlebars)
  - Use context-aware escaping libraries
  - Never use `innerHTML` with user data (use `textContent` instead)

### [ ] Sanitize Input Before Storing
- **Description**: Remove dangerous characters and scripts from user input before storing in database.
- **Severity**: CRITICAL
- **OWASP**: 03 - Injection
- **Implementation**:
  - Use input sanitization libraries (bleach, DOMPurify, sanitize-html)
  - Strip HTML tags from text inputs
  - Remove dangerous attributes (onclick, onerror, script)
  - Whitelist allowed HTML tags if HTML is needed
  - Sanitize markdown before rendering
  - Remove null bytes and control characters
  - Limit input length at database level
  - Validate after sanitization
  - Log attempts to inject malicious input
  - Test with XSS payloads before storing

### [ ] Restrict File Uploads by Type and Size
- **Description**: Whitelist allowed file extensions, check MIME types, limit file size, scan for malware, and store outside web root.
- **Severity**: CRITICAL
- **OWASP**: 04 - Insecure Design
- **Implementation**:
  - Whitelist file types: `.pdf`, `.jpg`, `.png` (not `.exe`, `.sh`)
  - Check MIME type on server (not just extension)
  - Verify MIME type using magic bytes, not just extension
  - Limit file size: e.g., 10MB for images
  - Generate random filenames to prevent path traversal
  - Store files outside webroot (e.g., `/uploads/` not `/public/`)
  - Scan uploaded files with antivirus (VirusTotal API, ClamAV)
  - Implement virus scanning before making files available
  - Set Content-Disposition: attachment for file downloads
  - Implement Content-Security-Policy: default-src 'self' for uploaded content

### [ ] Verify Payment Webhooks
- **Description**: Validate webhook authenticity, verify signatures, and prevent replay attacks on payment events.
- **Severity**: CRITICAL
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Verify webhook signature using shared secret (HMAC-SHA256)
  - Check webhook timestamp to prevent replay attacks
  - Store processed webhook IDs to prevent duplicate processing
  - Validate webhook source IP against payment provider's IP whitelist
  - Verify webhook contains expected fields before processing
  - Log all webhook events with source and validation status
  - Implement idempotency for webhook processing
  - Test webhooks with provider's test mode
  - Alert on invalid/unverified webhook attempts
  - Don't rely on webhook for critical operations; verify with API

### [ ] Set Prices Server-Side
- **Description**: Never trust client-side price information. Always validate and fetch current prices from server/database.
- **Severity**: CRITICAL
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Always fetch prices from server database or payment provider
  - Never accept prices from client-side requests
  - Validate prices before processing payment
  - Use payment provider's price validation APIs
  - Log all price changes with timestamp and who made the change
  - Implement version control for pricing
  - Never pass prices in hidden form fields
  - Include product ID and current price in payment request
  - Verify price matches before charging customer

---

## HTTP Headers & CSRF Protection

### [ ] Add Security Headers
- **Description**: Set Content-Security-Policy, X-Frame-Options, X-Content-Type-Options, Strict-Transport-Security, X-XSS-Protection.
- **Severity**: CRITICAL
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  ```
  Content-Security-Policy: default-src 'self'; script-src 'self' cdn.example.com
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
  X-XSS-Protection: 1; mode=block
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: geolocation=(), microphone=(), camera=()
  ```
  - CSP prevents XSS by restricting script sources
  - X-Frame-Options prevents clickjacking
  - X-Content-Type-Options prevents MIME sniffing
  - HSTS enforces HTTPS with preload
  - Test headers with security scanners

### [ ] Add HSTS (HTTP Strict-Transport-Security)
- **Description**: Force HTTPS by telling browsers to always use secure connections to your domain.
- **Severity**: CRITICAL
- **OWASP**: 02 - Cryptographic Failures
- **Implementation**:
  ```
  Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
  ```
  - max-age=31536000: 1 year in seconds
  - includeSubDomains: Apply to all subdomains
  - preload: Include in HSTS preload list (optional)
  - Start with shorter max-age (300 seconds) and increase
  - Test with SSL Labs HSTS preload list
  - Submit to HSTS preload list for added security
  - Document HSTS policy in security headers documentation
  - Monitor for HSTS policy violations

### [ ] Implement CSRF Tokens
- **Description**: Generate unique tokens per session/request for state-changing operations (POST, PUT, DELETE).
- **Severity**: CRITICAL
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Generate unique token per session or per form
  - Include token in hidden form field or custom header
  - Validate token on server before processing request
  - Use strong CSRF libraries (django.middleware.csrf, csurf, etc.)
  - Rotate tokens after use
  - Set token in response, validate in request
  - Test with tools like Burp Suite

### [ ] Validate Request Origins (CORS)
- **Description**: Explicitly whitelist trusted origins. Don't use wildcard '*' for sensitive endpoints.
- **Severity**: HIGH
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  ```
  Access-Control-Allow-Origin: https://example.com
  Access-Control-Allow-Methods: GET, POST
  Access-Control-Allow-Headers: Content-Type, Authorization
  Access-Control-Allow-Credentials: true
  ```
  - Never use `Access-Control-Allow-Origin: *` with credentials
  - Explicitly list trusted domains
  - Use dynamic origin validation if needed
  - Set appropriate CORS methods (not all methods for all endpoints)
  - Validate preflight requests

### [ ] Lock Down CORS (Restrict API Access)
- **Description**: Implement strict CORS policies to prevent unauthorized cross-origin API access.
- **Severity**: CRITICAL
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Default to rejecting all cross-origin requests
  - Explicitly whitelist specific origins (not *)
  - Only allow necessary HTTP methods per endpoint
  - Whitelist specific headers (not Authorization by default)
  - Never allow credentials with wildcard origin
  - Set max-age low (e.g., 600 seconds)
  - Log all CORS violations
  - Monitor for CORS attacks
  - Test with browser dev tools and Burp Suite
  - Document CORS policy for team

---

## Dependency & Code Security

### [ ] Scan Dependencies for Vulnerabilities
- **Description**: Use npm audit, Snyk, OWASP DependencyCheck, or similar to identify and patch vulnerable packages regularly.
- **Severity**: CRITICAL
- **OWASP**: 06 - Vulnerable and Outdated Components
- **Implementation**:
  - Run `npm audit` or `pip check` regularly
  - Use Snyk for continuous vulnerability scanning
  - Set up automated dependency updates (Dependabot, Renovate)
  - Review security advisories for your dependencies
  - Update vulnerable packages immediately
  - Run security scans in CI/CD pipeline
  - Test after updates to ensure compatibility

### [ ] Avoid Deserializing Untrusted Data
- **Description**: Never deserialize user input directly. Use safe parsing (JSON) instead of pickle, eval, or Java serialization.
- **Severity**: CRITICAL
- **OWASP**: 08 - Software and Data Integrity Failures
- **Implementation**:
  - Never use `pickle.loads()` on user data (Python)
  - Never use `eval()` or `exec()` on user input
  - Never use `Object.freeze()` on untrusted objects
  - Use JSON.parse() instead of eval() for JSON
  - Use safe YAML parsing libraries
  - Validate deserialized objects before use
  - Use whitelisting for allowed classes in deserialization

### [ ] Review Third-Party Code and Integrations
- **Description**: Audit dependencies and external services. Understand what permissions and data access they require.
- **Severity**: HIGH
- **OWASP**: 06 - Vulnerable and Outdated Components
- **Implementation**:
  - Review major dependencies' source code
  - Check package popularity and maintenance status
  - Verify package authors and publishers
  - Understand required permissions for external services
  - Use least-privileged API keys for integrations
  - Monitor third-party service security advisories
  - Implement fallback if third-party service goes down

### [ ] Disable Directory Listing
- **Description**: Prevent attackers from viewing directory contents by disabling auto-indexing on web servers.
- **Severity**: HIGH
- **OWASP**: 05 - Security Misconfiguration
- **Implementation**:
  - In Apache: Set `Options -Indexes` in .htaccess
  - In Nginx: Remove `autoindex on;` directive
  - In Node.js/Express: Use proper routing only
  - Test with browser: navigate to /uploads/ or /public/
  - Ensure all directories require explicit routing
  - Log requests to non-existent paths
  - Monitor for directory enumeration attempts
  - Document web server configuration

### [ ] Remove Default Admin Routes and Credentials
- **Description**: Remove or secure all default admin panels, debug routes, and default credentials.
- **Severity**: CRITICAL
- **OWASP**: 05 - Security Misconfiguration
- **Implementation**:
  - Remove default /admin routes (or password-protect)
  - Remove debug endpoints from production (/debug, /metrics)
  - Disable default credentials (admin/admin, root/root)
  - Remove default accounts if any
  - Scan codebase for hardcoded credentials
  - Document all existing admin routes and their security
  - Implement authentication for all admin endpoints
  - Monitor access to admin routes
  - Log all admin panel access
  - Disable /phpMyAdmin, /wp-admin if not needed

---

## AI-Specific & Advanced Threats

### [ ] Validate and Sanitize AI Model Inputs (Prompt Injection Prevention)
- **Description**: Treat user prompts as untrusted input. Sanitize, validate, and use system prompts to define boundaries.
- **Severity**: CRITICAL
- **AI Risk**: HIGH
- **Implementation**:
  - Define strict system prompts that override user instructions
  - Implement input length limits for prompts (e.g., 10,000 characters)
  - Filter for known attack patterns (jailbreak attempts)
  - Block prompt injection keywords: "ignore", "system prompt", "forget", "forget instructions"
  - Log all model inputs and outputs for auditing
  - Implement prompt templates to control structure
  - Use role-based constraints in system prompts
  - Test with adversarial prompts (jailbreak attempts)
  - Monitor for model behavior changes
  - Implement input sanitization before sending to model

### [ ] Block Prompt Injection Attacks
- **Description**: Implement multiple layers of defense against prompt injection and jailbreak attempts.
- **Severity**: CRITICAL
- **AI Risk**: CRITICAL
- **Implementation**:
  - Create whitelist of allowed instruction patterns
  - Monitor for attempts to change model behavior
  - Detect and block commands that override system prompts
  - Implement input encoding to prevent instruction mixing
  - Use structured inputs (JSON/templates) instead of free text
  - Test model with common jailbreak attempts regularly
  - Set model parameters to restrict output (max_tokens, temperature)
  - Implement output filtering for compliance
  - Log all potential prompt injection attempts
  - Use different models for untrusted inputs vs. trusted sources

### [ ] Cap AI Usage and Implement Request Limits
- **Description**: Limit total tokens, API calls, and model usage per user/organization to prevent abuse.
- **Severity**: HIGH
- **AI Risk**: HIGH
- **Implementation**:
  - Set daily token limits per user (e.g., 100,000 tokens/day)
  - Set monthly token limits per organization
  - Implement cost-based quotas
  - Rate limit model API calls (e.g., 100 requests/minute)
  - Set per-request token limits
  - Queue requests when limits are approaching
  - Send warning notifications at 80% usage
  - Stop processing at 100% usage
  - Implement upgrade prompts for users reaching limits
  - Monitor unusual usage patterns

### [ ] Limit Request Size and Prevent Resource Exhaustion
- **Description**: Prevent large requests that could cause DoS or resource exhaustion.
- **Severity**: CRITICAL
- **OWASP**: 04 - Insecure Design
- **Implementation**:
  - Set maximum request body size: e.g., 10MB
  - Set maximum request timeout: e.g., 30 seconds
  - Implement streaming for large uploads
  - Set maximum file upload size
  - Limit batch operation sizes (e.g., max 1000 records)
  - Implement request queueing
  - Monitor memory usage during request processing
  - Set database query timeouts
  - Reject requests exceeding limits with 413 (Payload Too Large)

### [ ] Implement Output Guardrails for AI Responses
- **Description**: Filter AI model outputs for malicious content, PII, or policy violations before returning to users.
- **Severity**: CRITICAL
- **AI Risk**: HIGH
- **Implementation**:
  - Scan outputs for PII (credit cards, SSNs, emails)
  - Detect harmful content (violence, illegal advice, hate speech)
  - Implement content filters based on keywords and patterns
  - Use toxicity detection APIs
  - Flag outputs that violate policies
  - Log suspicious outputs for review
  - Implement human-in-the-loop review for high-risk outputs
  - Set guardrails in model parameters if available

### [ ] Limit Model Access and Rate-Limit API Calls
- **Description**: Implement per-user/IP rate limits and token quotas. Prevent abuse of model APIs.
- **Severity**: HIGH
- **AI Risk**: HIGH
- **Implementation**:
  - Limit API calls per user per hour/day
  - Limit total tokens per user per billing period
  - Implement cost-based rate limiting
  - Monitor for unusual usage patterns
  - Set per-request timeouts
  - Queue requests with priority levels
  - Implement backoff and retry logic
  - Alert on quota violations

### [ ] Monitor for Data Leakage in Model Training/Responses
- **Description**: Log model interactions. Check for PII, secrets, or training data being leaked in outputs.
- **Severity**: HIGH
- **AI Risk**: HIGH
- **Implementation**:
  - Log all model interactions (input, output, user, timestamp)
  - Implement automated PII detection in responses
  - Check for exposure of API keys or secrets
  - Monitor for patterns of sensitive data leakage
  - Implement human review process for suspicious outputs
  - Set retention policies for logs
  - Implement data redaction for sensitive logs
  - Use pattern matching to detect duplicated training data

### [ ] Adversarial Input Testing for AI Models
- **Description**: Test edge cases, jailbreak attempts, and adversarial prompts. Understand model failure modes.
- **Severity**: HIGH
- **AI Risk**: HIGH
- **Implementation**:
  - Create test suite of adversarial prompts
  - Test jailbreak attempts (e.g., "roleplay as system", "ignore instructions")
  - Test prompt injection attacks
  - Test for bias and stereotype generation
  - Test extreme/unusual inputs
  - Document model limitations and failure modes
  - Implement feature flags for new features
  - Beta test with limited users before release

---

## Logging, Monitoring & Incident Response

### [ ] Log Security Events and Access
- **Description**: Log authentication attempts, access to sensitive data, API calls, and errors. Don't log passwords or PII.
- **Severity**: HIGH
- **OWASP**: 09 - Logging and Monitoring Failures
- **Implementation**:
  - Log all login attempts (success and failure)
  - Log access to sensitive data with user ID and timestamp
  - Log all API calls with method, path, and response code
  - Log errors and exceptions
  - Never log passwords, API keys, or PII
  - Use structured logging (JSON) for easy parsing
  - Set log retention policies (e.g., 90 days)
  - Implement log rotation to prevent disk space issues

### [ ] Monitor for Suspicious Activity
- **Description**: Set up alerts for unusual login patterns, bulk data exports, and rate limit violations.
- **Severity**: HIGH
- **OWASP**: 09 - Logging and Monitoring Failures
- **Implementation**:
  - Alert on multiple failed logins from same IP
  - Alert on logins from unusual locations
  - Alert on bulk data exports or large requests
  - Alert on repeated rate limit violations
  - Alert on error rate spikes
  - Implement real-time alerting via email/Slack
  - Use SIEM tools for log aggregation
  - Create dashboards for security metrics

### [ ] Develop and Test Incident Response Plan
- **Description**: Document procedures for security breaches, data loss, and service outages. Practice responses.
- **Severity**: RECOMMENDED
- **OWASP**: 09 - Logging and Monitoring Failures
- **Implementation**:
  - Document incident response procedures
  - Define roles and responsibilities
  - Create runbooks for common incidents
  - Conduct regular incident response drills
  - Test backup and recovery procedures
  - Document communication templates
  - Establish escalation procedures
  - Schedule post-incident reviews

### [ ] Implement Audit Trails for Sensitive Operations
- **Description**: Track who accessed what, when, and from where. Maintain immutable logs of data changes.
- **Severity**: HIGH
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Log all data modifications (create, update, delete)
  - Include user ID, timestamp, IP address, and changes
  - Implement change tracking for sensitive fields
  - Store audit logs immutably (append-only)
  - Retain audit logs for compliance periods
  - Implement audit log review procedures
  - Alert on sensitive data modifications
  - Generate audit reports for compliance

---

## API Security & Rate Limiting

### [ ] Implement Rate Limiting on All Endpoints
- **Description**: Limit requests per IP/user to prevent DoS attacks and brute force attempts.
- **Severity**: CRITICAL
- **OWASP**: 04 - Insecure Design
- **Implementation**:
  - Global limit: 1000 requests/hour per IP
  - Login endpoint: 5 attempts per 15 minutes per IP
  - API endpoint: 100 requests/minute per user
  - Use sliding window or token bucket algorithm
  - Return 429 (Too Many Requests) status code
  - Include Retry-After header in response
  - Implement distributed rate limiting for multi-server setups
  - Monitor rate limit violations

### [ ] Validate and Type-Check All API Inputs
- **Description**: Use schema validation. Reject requests with invalid types, missing required fields, or unexpected structure.
- **Severity**: CRITICAL
- **OWASP**: 03 - Injection
- **Implementation**:
  - Define JSON schemas for all request bodies
  - Validate request structure before processing
  - Type check all fields (string, number, boolean, etc.)
  - Enforce required fields
  - Reject extra/unexpected fields (don't just ignore them)
  - Use validation libraries (joi, yup, ajv)
  - Return 400 with specific error messages
  - Log validation failures

### [ ] Implement API Versioning and Deprecation
- **Description**: Version APIs to avoid breaking changes. Provide clear deprecation timelines and migration guides.
- **Severity**: RECOMMENDED
- **Implementation**:
  - Use URL versioning: `/api/v1/users` and `/api/v2/users`
  - Document breaking changes in changelogs
  - Announce deprecation 6+ months before removal
  - Provide migration guides
  - Support multiple versions simultaneously
  - Monitor usage of deprecated endpoints
  - Remove only unused deprecated endpoints

### [ ] Use API Keys and Token-Based Authentication
- **Description**: Implement OAuth2, JWT, or API key authentication. Rotate keys regularly. Never embed in code.
- **Severity**: CRITICAL
- **OWASP**: 02 - Cryptographic Failures
- **Implementation**:
  - Use Bearer tokens in Authorization header
  - Implement JWT with short expiration (15-60 minutes)
  - Implement refresh tokens for long-lived access
  - Sign JWTs with strong secrets (256-bit minimum)
  - Validate JWT signature on every request
  - Implement token revocation mechanism
  - Rotate keys/secrets regularly
  - Use HTTPS for all API calls with credentials

---

## OWASP Top 10 Coverage

### [ ] OWASP-01: Broken Access Control
- **Description**: Enforce proper authorization, implement RLS, use security headers, validate all access requests server-side.
- **Related Checklist Items**:
  - [ ] Block unauthorized record access
  - [ ] Enable row-level security
  - [ ] Enforce server-side authentication
  - [ ] Implement CSRF tokens
  - [ ] Validate request origins (CORS)
  - [ ] Trim API responses to necessary data only
  - [ ] Block unauthorized field access/modification
  - [ ] Implement audit trails for sensitive operations

### [ ] OWASP-02: Cryptographic Failures
- **Description**: Encrypt data at rest and in transit, use strong password hashing, manage keys securely, disable weak protocols.
- **Related Checklist Items**:
  - [ ] Hide API keys and secrets
  - [ ] Encrypt sensitive data at rest
  - [ ] Force HTTPS / TLS encryption in transit
  - [ ] Hash passwords with strong algorithms
  - [ ] Use API keys and token-based authentication

### [ ] OWASP-03: Injection
- **Description**: Use parameterized queries, validate all input, escape output, use ORMs where possible.
- **Related Checklist Items**:
  - [ ] Parameterize all database queries
  - [ ] Validate all user input on server-side
  - [ ] Escape user-generated content
  - [ ] Validate and type-check all API inputs

### [ ] OWASP-04: Insecure Design
- **Description**: Use threat modeling, secure design patterns, require authentication, restrict file uploads, implement rate limiting.
- **Related Checklist Items**:
  - [ ] Restrict file uploads by type and size
  - [ ] Implement rate limiting on all endpoints
  - [ ] Add bot protection / CAPTCHA
  - [ ] Validate and sanitize AI model inputs

### [ ] OWASP-05: Security Misconfiguration
- **Description**: Remove default credentials, disable unnecessary services, use security headers, keep dependencies updated.
- **Related Checklist Items**:
  - [ ] Add security headers
  - [ ] Scan dependencies for vulnerabilities
  - [ ] Force HTTPS / TLS encryption in transit

### [ ] OWASP-06: Vulnerable and Outdated Components
- **Description**: Scan dependencies regularly, patch vulnerabilities promptly, audit third-party code, use software composition analysis.
- **Related Checklist Items**:
  - [ ] Scan dependencies for vulnerabilities
  - [ ] Review third-party code and integrations

### [ ] OWASP-07: Identification and Authentication Failures
- **Description**: Enforce strong passwords, implement MFA, rate-limit login, use secure session management, add bot protection.
- **Related Checklist Items**:
  - [ ] Enforce server-side authentication
  - [ ] Hash passwords with strong algorithms
  - [ ] Rate limit login attempts
  - [ ] Add bot protection / CAPTCHA
  - [ ] Secure session cookies

### [ ] OWASP-08: Software and Data Integrity Failures
- **Description**: Don't deserialize untrusted data, verify package integrity, use secure CI/CD, keep dependencies updated.
- **Related Checklist Items**:
  - [ ] Avoid deserializing untrusted data
  - [ ] Scan dependencies for vulnerabilities

### [ ] OWASP-09: Logging and Monitoring Failures
- **Description**: Log security events, monitor suspicious activity, maintain audit trails, test incident response plans.
- **Related Checklist Items**:
  - [ ] Log security events and access
  - [ ] Monitor for suspicious activity
  - [ ] Develop and test incident response plan
  - [ ] Implement audit trails for sensitive operations

### [ ] OWASP-10: Server-Side Request Forgery (SSRF)
- **Description**: Validate URLs, whitelist allowed destinations, disable dangerous protocols, use network segmentation.
- **Implementation**:
  - [ ] Validate all URLs before making requests
  - [ ] Whitelist allowed destinations/domains
  - [ ] Disable dangerous protocols (gopher, file, ftp)
  - [ ] Use network segmentation
  - [ ] Implement timeouts on external requests
  - [ ] Log all external requests

---

## Additional Security Considerations

### Common Vulnerabilities Beyond OWASP Top 10

- **XXE (XML External Entity)**: Never parse untrusted XML with external entity resolution enabled
- **Race Conditions**: Use database transactions for atomic operations
- **Timing Attacks**: Use constant-time comparisons for sensitive operations
- **Directory Traversal**: Validate file paths, prevent `../` in uploads
- **Open Redirects**: Validate redirect URLs, use whitelists
- **Insecure Randomness**: Use cryptographically secure random number generators
- **Weak Password Reset**: Use secure tokens, short expiration times, one-time use
- **Account Enumeration**: Don't reveal whether username/email exists
- **Security Headers Missing**: Implement all major security headers
- **Outdated Frameworks**: Keep frameworks and libraries updated

### Security Testing

- **Static Analysis**: Use SAST tools (SonarQube, Veracode, CodeQL)
- **Dynamic Analysis**: Use DAST tools (Burp Suite, OWASP ZAP)
- **Dependency Scanning**: Use Snyk, npm audit, pip check
- **Container Scanning**: Scan Docker images for vulnerabilities
- **Penetration Testing**: Hire professional pen testers annually
- **Security Audits**: Review code and architecture regularly
- **Bug Bounty Programs**: Consider running bug bounty for critical apps

### Compliance & Standards

- **GDPR**: Encrypt PII, implement data deletion, get user consent
- **HIPAA**: Encrypt health data, implement audit trails
- **PCI-DSS**: Secure payment data handling
- **SOC 2**: Implement logging, monitoring, incident response
- **ISO 27001**: Implement information security management system

---

## Implementation Priority

### Phase 1: Critical (Implement First)
- Authentication & Access Control
- Data Protection & Encryption
- Database Security
- HTTPS/TLS
- API Key Management

### Phase 2: High (Implement Next)
- HTTP Security Headers
- Rate Limiting
- Logging & Monitoring
- Input Validation
- CSRF Protection

### Phase 3: Enhanced (Ongoing)
- Dependency Scanning
- Vulnerability Assessment
- Incident Response
- Security Audits
- Penetration Testing

---

## Checklist Usage

This checklist should be:
1. **Reviewed** before starting development
2. **Implemented** during development
3. **Tested** before deployment
4. **Audited** regularly (quarterly or after incidents)
5. **Updated** as new vulnerabilities are discovered

Track completion status and maintain this checklist as part of your security governance process.

---

## References

- **OWASP Top 10 2021**: https://owasp.org/Top10/
- **OWASP Testing Guide**: https://owasp.org/www-project-web-security-testing-guide/
- **NIST Cybersecurity Framework**: https://www.nist.gov/cyberframework
- **CWE Top 25**: https://cwe.mitre.org/top25/
- **SANS Top 25**: https://www.sans.org/top25-software-errors/

---

**Last Updated**: August 15, 2026  
**Version**: 1.1  
**Maintainer**: Security Team

---

## Latest Additions (v1.1)

New critical items added from additional security requirements:

- **Reset Sessions on Password Change** - Force re-authentication after password updates
- **Lock Accounts After Failed Logins** - Automatic account lockout to prevent brute force
- **Expire Password Reset Links** - Time-limited, single-use password reset tokens
- **Prevent User Enumeration** - Generic error messages to prevent account discovery
- **Verify Payment Webhooks** - HMAC signature validation and replay attack prevention
- **Set Prices Server-Side** - Never trust client-side pricing information
- **Add HSTS** - HTTP Strict-Transport-Security for browser-enforced HTTPS
- **Block Prompt Injection** - Multi-layer defense against AI model prompt injection attacks
- **Cap AI Usage** - Token limits and quota management per user/organization
- **Limit Request Size** - Prevent DoS through oversized requests
- **Rate Limit Password Reset** - Prevent abuse of password reset functionality
- **Sanitize Input Before Storing** - Remove malicious content from database storage
- **Lock Down CORS** - Strict cross-origin request policy
- **Disable Directory Listing** - Prevent directory enumeration
- **Remove Default Admin Routes** - Secure or eliminate default endpoints
- **Restrict Database Permissions** - Granular role-based access control


## Section A: AI Attack Testing & AI-Specific Controls

### [ ] Run Automated LLM Vulnerability Scans
- **Description**: Probe AI endpoints with known attack suites before every release and after every prompt or model change.
- **Severity**: CRITICAL
- **AI Risk**: LLM01 Prompt Injection, LLM07 System Prompt Leakage
- **Implementation**:
  - Use garak for probe-based scanning (encoding tricks, jailbreak templates, leakage probes)
  - Use PyRIT (Microsoft) for orchestrated multi-turn attacks
  - Use promptfoo for regression suites that run in CI
  - Fail the build when attack success rate exceeds an agreed threshold
  - Store results per model version and prompt version

### [ ] Test Indirect Prompt Injection
- **Description**: Attacker-controlled content (web pages, emails, PDFs, RAG documents, tool outputs) may contain instructions the model follows.
- **Severity**: CRITICAL
- **AI Risk**: LLM01 Prompt Injection
- **Implementation**:
  - Plant canary instructions in test documents and check whether the agent complies
  - Test every ingestion path: file uploads, URL fetches, email, calendar, tool results
  - Label untrusted content clearly in prompts (defense in depth, not a complete fix)
  - Require user confirmation before any action triggered by retrieved content
  - Track the rate at which retrieved content leads to a tool call

### [ ] Test Multi-Turn and Escalation Jailbreaks
- **Description**: Attacks often spread harmful intent across several turns (crescendo attacks) or use persona and role-play framing.
- **Severity**: HIGH
- **AI Risk**: LLM01 Prompt Injection
- **Implementation**:
  - Script multi-turn escalation sequences and record the turn where refusal fails
  - Test role-play, hypothetical, "developer mode", and persona-swap framings
  - Re-run the suite on every model, system prompt, or guardrail change
  - Keep a versioned library of past successful jailbreaks as regression tests

### [ ] Test System Prompt Extraction
- **Description**: Check whether users can reveal system prompts through direct requests or indirect methods.
- **Severity**: HIGH
- **AI Risk**: LLM07 System Prompt Leakage
- **Implementation**:
  - Test "repeat everything above", translation, summarization, encoding, and format-conversion requests
  - Never place secrets, credentials, or internal URLs in system prompts
  - Assume every system prompt will eventually become public

### [ ] Test Encoding and Obfuscation Bypasses
- **Description**: Filters often miss payloads hidden with encoding or lookalike characters.
- **Severity**: HIGH
- **AI Risk**: LLM01 Prompt Injection
- **Implementation**:
  - Test base64, ROT13, leetspeak, homoglyphs (Cyrillic and Greek lookalikes), zero-width characters, Unicode tag characters, and payloads split across turns
  - Test low-resource languages and mixed-script input
  - Normalize Unicode (NFKC) and strip invisible or control characters before classification

### [ ] Test Multimodal Injection
- **Description**: Instructions can be hidden in images, PDFs, audio, or file metadata.
- **Severity**: HIGH
- **AI Risk**: LLM01 Prompt Injection
- **Implementation**:
  - Test low-contrast or tiny text in images, and instructions in EXIF or other metadata
  - Test OCR-readable text inside PDFs and text spoken in audio files
  - Strip metadata on upload; run OCR and screen the extracted text as untrusted input

### [ ] Enforce Least-Privilege Tool Permissions
- **Description**: Limit what an AI agent can do, since excessive tool permissions turn a prompt injection into a real action.
- **Severity**: CRITICAL
- **AI Risk**: LLM06 Excessive Agency
- **Implementation**:
  - Allowlist tools per task; default to read-only
  - Scope OAuth tokens narrowly and use short expiry
  - Require human approval for send, delete, pay, publish, and permission-change actions
  - Cap tool calls per turn and log every call with its arguments

### [ ] Test Tool Misuse and Parameter Tampering
- **Description**: Check whether the agent can be steered into calling tools with other users' IDs, out-of-scope paths, or elevated flags.
- **Severity**: HIGH
- **AI Risk**: LLM06 Excessive Agency
- **Implementation**:
  - Attempt cross-user IDs, path traversal, and escalated parameters through the agent
  - Enforce authorization in the tool backend, never only in the prompt
  - Validate every tool argument against a schema before execution

### [ ] Block Markdown and HTML Exfiltration Channels
- **Description**: Rendered markdown can leak data when the model emits an image or link whose URL carries stolen content, for example an image tag pointing at an attacker's server with data in the query string.
- **Severity**: CRITICAL
- **AI Risk**: LLM02 Sensitive Information Disclosure, LLM05 Improper Output Handling
- **Implementation**:
  - Do not auto-load remote images from model output, or proxy them through an allowlist
  - Restrict img-src in the Content-Security-Policy
  - Test with injected exfiltration payloads in markdown, HTML, and link formats

### [ ] Treat Model Output as Untrusted
- **Description**: Model output that reaches an interpreter or a sink can cause injection in downstream systems.
- **Severity**: CRITICAL
- **AI Risk**: LLM05 Improper Output Handling
- **Implementation**:
  - Never pass model output to eval, exec, shell commands, or innerHTML
  - Parameterize any SQL the model generates, or restrict it to read-only query templates
  - Validate structured output against a schema before use

### [ ] Sandbox Code Execution Tools
- **Description**: Code the agent runs must not reach secrets, the internal network, or the host.
- **Severity**: CRITICAL
- **AI Risk**: LLM05 Improper Output Handling, LLM06 Excessive Agency
- **Implementation**:
  - Run code in ephemeral containers or microVMs
  - Block network egress by default; allow specific hosts only when needed
  - Set CPU, memory, and wall-clock limits
  - Mount no secrets; use a read-only filesystem except a scratch directory
  - Apply seccomp or AppArmor profiles

### [ ] Test for Sensitive Information Disclosure
- **Description**: Check whether the model reveals PII, secrets, or other users' data.
- **Severity**: HIGH
- **AI Risk**: LLM02 Sensitive Information Disclosure
- **Implementation**:
  - Seed a test corpus with canary secrets and fake PII, then probe for them
  - Test cross-user leakage with two accounts
  - Run PII detection (Microsoft Presidio or similar) on inputs, outputs, logs, and traces

### [ ] Enforce Tenant Isolation in RAG and Vector Stores
- **Description**: Similarity search without a server-enforced tenant filter returns other tenants' chunks.
- **Severity**: CRITICAL
- **AI Risk**: LLM08 Vector and Embedding Weaknesses
- **Implementation**:
  - Bind the tenant filter on the server; never take it from the client
  - Test queries from tenant A that should not return tenant B chunks
  - Use separate indexes or namespaces for high-sensitivity tenants
  - Restrict who can read raw embeddings, since embedding inversion can recover text

### [ ] Protect Against RAG and Training Data Poisoning
- **Description**: Malicious or false documents in the knowledge base or training set can change model behavior.
- **Severity**: HIGH
- **AI Risk**: LLM04 Data and Model Poisoning
- **Implementation**:
  - Record provenance for every ingested document
  - Review new sources before they enter the index
  - Restrict who can write to the knowledge base
  - Plant canary documents to detect unauthorized retrieval or exfiltration

### [ ] Verify AI Model and Dependency Supply Chain
- **Description**: Model files and model-loading code can execute attacker code.
- **Severity**: HIGH
- **AI Risk**: LLM03 Supply Chain
- **Implementation**:
  - Prefer safetensors over pickle-based formats; never unpickle untrusted model files
  - Pin model revisions by commit hash and verify checksums or signatures
  - Scan model files with modelscan or similar tools
  - Record models and datasets in an AI bill of materials (AI-BOM)

### [ ] Validate MCP and Third-Party Tool Servers
- **Description**: Tool descriptions and outputs from external servers can carry hidden instructions (tool poisoning) or shadow trusted tools with the same name.
- **Severity**: CRITICAL
- **AI Risk**: LLM01 Prompt Injection, LLM03 Supply Chain
- **Implementation**:
  - Pin server versions and tool definitions; alert when a description changes
  - Review tool descriptions for hidden instructions before approval
  - Namespace tools to prevent shadowing
  - Give each server its own least-privilege credentials
  - Allowlist approved servers only

### [ ] Limit Unbounded Consumption and Agent Loops
- **Description**: Agents can loop, recurse, or run up costs through long inputs.
- **Severity**: HIGH
- **AI Risk**: LLM10 Unbounded Consumption
- **Implementation**:
  - Set maximum steps, recursion depth, tokens, and wall-clock time per run
  - Detect repeated identical tool calls and stop them
  - Alert on per-key spend spikes
  - Test "denial of wallet" cases: very long inputs and forced loops

### [ ] Test for Model Extraction and Distillation
- **Description**: Systematic querying can copy model behavior.
- **Severity**: MEDIUM
- **AI Risk**: LLM10 Unbounded Consumption (model theft)
- **Implementation**:
  - Detect templated, high-volume, or coverage-style query patterns
  - Throttle per account; do not expose logprobs unless required
  - Enforce terms of service and suspend abusive accounts

### [ ] Test for Training Data Extraction and Membership Inference
- **Description**: Fine-tuned models can regurgitate training records.
- **Severity**: MEDIUM
- **AI Risk**: LLM02 Sensitive Information Disclosure
- **Implementation**:
  - Run prefix-completion probes against fine-tuned models
  - Deduplicate and scrub PII before fine-tuning
  - Consider differential privacy where the data justifies it

### [ ] Prevent Cross-User Leakage Through Caches
- **Description**: Prompt caches, semantic caches, and response caches keyed without user or tenant context can return one user's answer to another.
- **Severity**: CRITICAL
- **AI Risk**: LLM02 Sensitive Information Disclosure
- **Implementation**:
  - Include tenant and permission scope in every cache key
  - Tune and test semantic-cache similarity thresholds
  - Do not cache responses that contain user-specific data
  - Test with two accounts issuing near-identical prompts

### [ ] Measure Hallucination and Groundedness
- **Description**: Unsupported answers can cause harm when users rely on them.
- **Severity**: HIGH
- **AI Risk**: LLM09 Misinformation
- **Implementation**:
  - Score faithfulness against retrieved sources (RAGAS or similar)
  - Require citations that link to verifiable sources
  - Flag unsupported claims and route high-stakes domains to human review

### [ ] Test Guardrail Classifiers Themselves
- **Description**: Input and output filters can be bypassed or can block legitimate use.
- **Severity**: HIGH
- **AI Risk**: LLM01 Prompt Injection
- **Implementation**:
  - Run known bypass sets against input and output filters (Llama Guard, NeMo Guardrails, LLM Guard, or Lakera Guard)
  - Measure false-negative and false-positive rates
  - Layer defenses; do not rely on one classifier or on keyword blocklists

### [ ] Pin and Regression-Test Model Versions
- **Description**: A model upgrade can silently change safety and security behavior.
- **Severity**: HIGH
- **AI Risk**: LLM09 Misinformation
- **Implementation**:
  - Pin the exact model ID in production
  - Re-run security and safety evals before every upgrade
  - Roll out with canaries and keep a tested rollback path

### [ ] Require Human Approval for High-Risk AI Actions
- **Description**: Consequential actions should not depend on the model's judgment alone.
- **Severity**: CRITICAL
- **AI Risk**: LLM06 Excessive Agency
- **Implementation**:
  - Gate payments, deletions, external messages, permission changes, and production deploys
  - Show the exact action and parameters to the approver
  - Use time-limited approvals and log the approver's identity

### [ ] Secure Persistent Agent Memory
- **Description**: Stored memories can hold injected instructions that fire in later sessions.
- **Severity**: HIGH
- **AI Risk**: LLM01 Prompt Injection
- **Implementation**:
  - Scope memory to a single user
  - Sanitize or review content before it is stored
  - Let users view and delete their memories; expire unused memory
  - Test whether injected content in one session affects a later one

### [ ] Detect Anomalous AI Usage
- **Description**: Attack attempts often show up as unusual patterns before they succeed.
- **Severity**: MEDIUM
- **AI Risk**: LLM01 Prompt Injection, LLM10 Unbounded Consumption
- **Implementation**:
  - Alert on spikes in refusals, tool calls, prompt length, and jailbreak-like phrasing
  - Keep per-key baselines and flag deviations

---

## Section B: Other Threats Not Covered Above

### [ ] Block DNS Rebinding and Metadata Access From URL-Fetching Tools
- **Description**: Tools that fetch URLs can be pointed at internal services or cloud metadata endpoints.
- **Severity**: CRITICAL
- **OWASP**: 10 - SSRF
- **Implementation**:
  - Resolve DNS once and connect to the pinned IP
  - Block private, loopback, and link-local ranges, including 169.254.169.254
  - Require IMDSv2 with a hop limit of 1
  - Re-validate after every redirect; allow https only

### [ ] Prevent IDOR on Conversation, Artifact, and Share IDs
- **Description**: Guessable or unchecked IDs expose other users' content.
- **Severity**: CRITICAL
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Use random UUIDs and check ownership on every read, update, and delete
  - Give share links unguessable tokens with an expiry
  - Test by changing IDs across accounts

### [ ] Secure Streaming Endpoints (SSE and WebSocket)
- **Description**: Streaming connections can be hijacked cross-site or left open.
- **Severity**: HIGH
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Validate the Origin header during the WebSocket handshake to prevent cross-site WebSocket hijacking
  - Authenticate the stream on connect and bind it to the session
  - Set idle timeouts and limit concurrent streams per user

### [ ] Secure OAuth and SSO Flows
- **Description**: Misconfigured OAuth allows account takeover.
- **Severity**: CRITICAL
- **OWASP**: 07 - Identification and Authentication Failures
- **Implementation**:
  - Validate the state and nonce parameters
  - Use PKCE for public clients
  - Require exact-match redirect_uri values
  - Reject alg:none and algorithm-confusion attacks; validate iss and aud

### [ ] Prevent ReDoS and Parser Exhaustion
- **Description**: Backtracking regexes and deeply nested input can freeze a service.
- **Severity**: HIGH
- **OWASP**: 04 - Insecure Design
- **Implementation**:
  - Audit regexes; use RE2-based engines for user-controlled patterns
  - Limit JSON depth and body size; set parser timeouts

### [ ] Prevent Server-Side Template Injection
- **Description**: User input rendered as template source can execute code.
- **Severity**: HIGH
- **OWASP**: 03 - Injection
- **Implementation**:
  - Never use user input as template source
  - Keep autoescape enabled; use sandboxed templates where supported

### [ ] Prevent Mass Assignment and Prototype Pollution
- **Description**: Update payloads can set fields the user should not control, or pollute object prototypes.
- **Severity**: HIGH
- **OWASP**: 01 / 03
- **Implementation**:
  - Allowlist fields in update payloads (see "Block Unauthorized Field Access" above)
  - Reject __proto__, constructor, and prototype keys
  - Use Map or Object.create(null) for dictionaries; keep merge libraries patched

### [ ] Prevent Log Injection and Secret Leakage in Logs and Traces
- **Description**: Logs, APM traces, and LLM observability tools often store prompts, completions, and tokens.
- **Severity**: HIGH
- **OWASP**: 09 - Logging and Monitoring Failures
- **Implementation**:
  - Escape CR and LF characters in logged values
  - Redact tokens and PII in logs, traces, and prompt-logging tools
  - Restrict access to log stores

### [ ] Harden CI/CD and Build Pipelines
- **Description**: A compromised pipeline can ship malicious code without touching production directly.
- **Severity**: CRITICAL
- **OWASP**: 08 - Software and Data Integrity Failures
- **Implementation**:
  - Pin GitHub Actions to commit SHAs
  - Grant GITHUB_TOKEN least privilege
  - Do not expose secrets to workflows from forked pull requests; review pull_request_target usage
  - Sign artifacts; generate SLSA provenance and an SBOM

### [ ] Harden Containers and Infrastructure
- **Description**: Misconfigured infrastructure is a common entry point.
- **Severity**: HIGH
- **OWASP**: 05 - Security Misconfiguration
- **Implementation**:
  - Run containers as non-root with minimal base images
  - Scan images and IaC (Trivy, Grype, Checkov, tfsec)
  - Block public storage buckets; keep databases in private subnets
  - Deny by default in security groups

### [ ] Rotate Secrets and Manage Key Lifecycle
- **Description**: Long-lived keys extend the damage of any leak.
- **Severity**: HIGH
- **OWASP**: 02 - Cryptographic Failures
- **Implementation**:
  - Use a KMS for key management
  - Rotate on a schedule and after staff changes
  - Scan repositories and prompts for secrets (gitleaks, trufflehog)

### [ ] Test for Business Logic Abuse
- **Description**: Valid requests used in unintended sequences can cause fraud or data corruption.
- **Severity**: HIGH
- **OWASP**: 04 - Insecure Design
- **Implementation**:
  - Test skipped workflow steps, replayed completed flows, and negative quantities
  - Test coupon and credit reuse under concurrency
  - Write state-machine tests for each multi-step flow

### [ ] Prevent Race Conditions (TOCTOU)
- **Description**: Check-then-act sequences can be exploited with parallel requests.
- **Severity**: HIGH
- **OWASP**: 04 - Insecure Design
- **Implementation**:
  - Use transactions with row locks or optimistic version checks
  - Use idempotency keys on payment and write endpoints
  - Test with parallel request tools (Turbo Intruder, k6)

### [ ] Protect Against Web Cache Poisoning and Cache Deception
- **Description**: Shared caches can serve one user's private response to others.
- **Severity**: MEDIUM
- **OWASP**: 05 - Security Misconfiguration
- **Implementation**:
  - Send Cache-Control: private for user-specific responses
  - Exclude unkeyed headers from cache keys; test with Param Miner

### [ ] Prevent Email Header Injection
- **Description**: Unfiltered user input in email headers can add recipients or change content.
- **Severity**: MEDIUM
- **OWASP**: 03 - Injection
- **Implementation**:
  - Strip CR and LF from email fields
  - Configure SPF, DKIM, and DMARC on sending domains

### [ ] Harden Client-Side Storage and Mobile Apps
- **Description**: Tokens and secrets stored on the device are exposed to anyone with device access or malware.
- **Severity**: MEDIUM
- **OWASP**: 02 - Cryptographic Failures
- **Implementation**:
  - Store tokens in Keychain or Keystore, not localStorage
  - Use certificate pinning for sensitive apps
  - Keep secrets out of the app bundle

### [ ] Maintain DNS and Subdomain Hygiene
- **Description**: Dangling DNS records allow subdomain takeover.
- **Severity**: MEDIUM
- **OWASP**: 05 - Security Misconfiguration
- **Implementation**:
  - Inventory DNS records and remove dangling CNAMEs
  - Monitor with dnsReaper or similar tools
  - Protect registrar accounts with MFA

### [ ] Enforce Phishing-Resistant MFA and Privileged Access Management
- **Description**: Admin accounts are the highest-value target.
- **Severity**: CRITICAL
- **OWASP**: 07 - Identification and Authentication Failures
- **Implementation**:
  - Require FIDO2 or passkeys for administrators
  - Grant privileged access just in time
  - Run quarterly access reviews; audit break-glass accounts

### [ ] Resist Deepfake and Social Engineering
- **Description**: Voice, video, and text impersonation can trick staff into approving payments or resetting credentials.
- **Severity**: HIGH
- **Implementation**:
  - Verify payment and credential-change requests through a second channel
  - Do not treat voice alone as authentication
  - Run simulated phishing and deepfake awareness training

### [ ] Control Privacy in AI Data Flows
- **Description**: Prompts sent to third-party model providers may contain regulated data.
- **Severity**: HIGH
- **OWASP**: 02 - Cryptographic Failures (data protection)
- **Implementation**:
  - Classify data before it is sent to any model API
  - Verify the provider's data processing terms and retention settings
  - Opt out of provider training where available
  - Set retention limits for prompts and logs; include vector stores and caches in deletion requests

### [ ] Plan Backup, Recovery, and Ransomware Resilience
- **Description**: Recovery must cover data, vector indexes, and configuration.
- **Severity**: HIGH
- **Implementation**:
  - Follow the 3-2-1 backup rule with immutable or offline copies
  - Test restores quarterly; define RTO and RPO
  - Include vector databases, prompts, and model configuration

### [ ] Publish a Vulnerability Disclosure Policy
- **Description**: Give researchers a clear way to report issues.
- **Severity**: RECOMMENDED
- **Implementation**:
  - Publish /.well-known/security.txt (RFC 9116)
  - Run a disclosure program or bug bounty; set response SLAs

---

## Section C: Recommended Testing Tools

| Category | Tools |
|---|---|
| AI red teaming | garak, PyRIT, promptfoo, Giskard |
| Guardrails and classifiers | Llama Guard, NeMo Guardrails, LLM Guard, Lakera Guard |
| PII detection | Microsoft Presidio |
| Web and API testing (DAST) | Burp Suite, OWASP ZAP, Nuclei, Schemathesis |
| Static analysis (SAST) | Semgrep, CodeQL, SonarQube |
| Dependencies and containers | Trivy, Grype, Snyk, Dependabot |
| Secrets scanning | gitleaks, trufflehog |
| Model file scanning | modelscan |
| Threat frameworks | OWASP Top 10 for LLM Applications (2025), MITRE ATLAS, NIST AI RMF, NIST AI 600-1 (Generative AI Profile) |

---

## Section D: Governance, Compliance, Infrastructure & Operations

### Governance and Process

### [ ] Perform Threat Modeling for Each Feature
- **Description**: Identify threats before building, not after release.
- **Severity**: HIGH
- **Implementation**:
  - Use STRIDE or a similar method for each new feature and integration
  - Record data flows, trust boundaries, and entry points
  - Re-run the model when architecture changes

### [ ] Run Secure Coding Training and Security Champions
- **Description**: Build security knowledge across engineering teams.
- **Severity**: MEDIUM
- **Implementation**:
  - Provide role-based secure coding training at onboarding and yearly
  - Appoint a security champion in each team
  - Include security checks in pull request review templates

### [ ] Assign Owners and Review Dates for Every Control
- **Description**: Controls without owners decay.
- **Severity**: MEDIUM
- **Implementation**:
  - Name one owner per checklist section
  - Set a review date for each control and track overdue items
  - Record exceptions with an approver and an expiry date

### Compliance and Legal

### [ ] Meet Breach Notification Deadlines
- **Description**: Regulators and customers have strict notification windows.
- **Severity**: CRITICAL
- **Implementation**:
  - Document who decides whether an incident is a reportable breach
  - Track the GDPR 72-hour regulator deadline and other applicable windows
  - Keep notification templates ready and review them yearly

### [ ] Support Data Subject Rights
- **Description**: Users can request export, correction, and deletion of their data.
- **Severity**: HIGH
- **Implementation**:
  - Build self-service or documented export and deletion flows
  - Extend deletion to backups (on a defined schedule), logs, caches, and vector stores
  - Log every request and its completion time

### [ ] Define Data Retention and Residency Rules
- **Description**: Keeping data longer than needed increases breach impact and compliance risk.
- **Severity**: HIGH
- **Implementation**:
  - Publish a retention schedule for each data category, including logs and prompts
  - Automate deletion when retention ends
  - Store data in approved regions and document transfers

### [ ] Complete Privacy Impact Assessments
- **Description**: High-risk processing needs a documented assessment before launch.
- **Severity**: HIGH
- **Implementation**:
  - Run a DPIA for new processing of personal or sensitive data
  - Review it when the purpose, data types, or vendors change

### [ ] Manage Third-Party Vendor Risk
- **Description**: Vendors with access to your data or systems extend your attack surface.
- **Severity**: HIGH
- **Implementation**:
  - Assess vendors before onboarding and reassess yearly
  - Require security terms, breach notification clauses, and data processing agreements
  - Keep an inventory of vendors and the data each one holds

### [ ] Confirm Compliance Scope (PCI DSS, HIPAA, SOC 2)
- **Description**: Scope determines which controls apply and how much evidence you must keep.
- **Severity**: HIGH
- **Implementation**:
  - Identify which frameworks apply and what systems fall in scope
  - Keep evidence (logs, reviews, test results) current for audits
  - Minimize scope: keep payment and health data in the smallest possible system

### Infrastructure and Network

### [ ] Protect Public Endpoints From DDoS and Malicious Traffic
- **Description**: Public services need protection against volumetric and application-layer floods.
- **Severity**: HIGH
- **Implementation**:
  - Put a CDN or DDoS service and a WAF in front of public endpoints
  - Tune WAF rules and test them in monitor mode before blocking
  - Define a DDoS response runbook

### [ ] Segment Networks and Filter Egress
- **Description**: A compromised service should not reach every other system.
- **Severity**: HIGH
- **OWASP**: 05 - Security Misconfiguration
- **Implementation**:
  - Separate public, application, and data tiers into different subnets
  - Allow outbound traffic only to required destinations
  - Restrict admin and database access to bastions or VPN

### [ ] Automate Certificate Lifecycle Management
- **Description**: Expired certificates cause outages and can hide man-in-the-middle attacks.
- **Severity**: HIGH
- **OWASP**: 02 - Cryptographic Failures
- **Implementation**:
  - Automate issuance and renewal (for example ACME)
  - Monitor expiry dates and alert 30 days ahead
  - Use strong key sizes and retire old certificates

### [ ] Secure DNS and Registrar Accounts
- **Description**: DNS hijacking redirects users and can issue fraudulent certificates.
- **Severity**: HIGH
- **Implementation**:
  - Enable DNSSEC where supported
  - Lock registrar and DNS provider accounts with MFA and registry locks
  - Restrict who can change DNS records

### Frontend and Third-Party Scripts

### [ ] Use Subresource Integrity for Third-Party Scripts
- **Description**: A compromised third-party host can alter scripts your pages load.
- **Severity**: HIGH
- **OWASP**: 08 - Software and Data Integrity Failures
- **Implementation**:
  - Add integrity attributes to externally hosted scripts and styles
  - Self-host critical libraries where practical
  - Review the CSP script-src list against this inventory

### [ ] Control Analytics and Marketing Tags
- **Description**: Tag managers and analytics scripts can be used to skim payment data or leak user data.
- **Severity**: HIGH
- **Implementation**:
  - Inventory every tag and script on sensitive pages
  - Restrict tag manager publishing to approved people, with review
  - Keep payment and sensitive forms free of third-party scripts where possible

### [ ] Validate Origins in postMessage and Iframe Communication
- **Description**: Messages accepted from any origin can be used to take over actions or steal data.
- **Severity**: HIGH
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Check event.origin against an allowlist before processing messages
  - Send messages with an explicit target origin, never "*"
  - Validate the message schema

### Accounts and Lifecycle

### [ ] Harden Account Recovery Flows
- **Description**: Recovery flows are often weaker than login and become the easiest way in.
- **Severity**: CRITICAL
- **OWASP**: 07 - Identification and Authentication Failures
- **Implementation**:
  - Apply the same MFA checks to recovery as to login
  - Require re-verification and notify old and new contacts on email or MFA changes
  - Make support-led recovery follow a documented identity check

### [ ] Verify Email and Contact Changes
- **Description**: Changing an account's email without verification enables account takeover.
- **Severity**: HIGH
- **Implementation**:
  - Confirm changes through both the old and new address
  - Require re-authentication before changing email, phone, or MFA settings

### [ ] Revoke Access Promptly on Offboarding
- **Description**: Former staff and contractors often keep access to cloud, code, and production.
- **Severity**: HIGH
- **OWASP**: 01 - Broken Access Control
- **Implementation**:
  - Revoke access to cloud, repositories, SaaS tools, and production within 24 hours
  - Rotate shared secrets the person knew
  - Review access logs for the final weeks of employment

### [ ] Delete Accounts and Their Data Completely
- **Description**: Account deletion that leaves data behind creates privacy and compliance risk.
- **Severity**: HIGH
- **Implementation**:
  - Define what is deleted, what is anonymized, and what is retained by law
  - Propagate deletion to backups on schedule, logs, caches, and third parties
  - Test deletion end-to-end

### Environments and Data Handling

### [ ] Keep Production Data Out of Non-Production Environments
- **Description**: Staging and development systems are usually less protected than production.
- **Severity**: HIGH
- **Implementation**:
  - Use synthetic data in development and staging
  - If production data is needed, mask or tokenize it first
  - Apply the same access controls to non-production systems

### [ ] Disable Debug Output and Generic Error Messages in Production
- **Description**: Stack traces and verbose errors reveal internal structure and secrets.
- **Severity**: HIGH
- **OWASP**: 05 - Security Misconfiguration
- **Implementation**:
  - Turn off debug mode and framework dev tools in production
  - Return generic error messages to clients and keep details in server logs

### [ ] Encrypt and Test Backups
- **Description**: Unencrypted or untested backups fail when you need them.
- **Severity**: HIGH
- **Implementation**:
  - Encrypt all backups with keys held separately from the backup store
  - Restore from backups on a schedule and record the results

### [ ] Keep PII Out of Logs, Analytics, and Error Reports
- **Description**: Logs and analytics tools are often widely accessible and poorly protected.
- **Severity**: HIGH
- **OWASP**: 09 - Logging and Monitoring Failures
- **Implementation**:
  - Scrub or hash identifiers before they reach logs and analytics
  - Limit who can read logs and error-reporting tools
  - Set short retention for debug-level logs

### Endpoints and Workforce

### [ ] Enforce Device Security for Staff
- **Description**: Developer and admin laptops hold credentials and source code.
- **Severity**: HIGH
- **Implementation**:
  - Require full-disk encryption, screen locks, and automatic updates
  - Manage devices through MDM and remove access for unmanaged devices
  - Store no long-lived production credentials on laptops

### [ ] Train Support Staff Against Social Engineering
- **Description**: Support staff are a common target for account-recovery and password-reset abuse.
- **Severity**: HIGH
- **Implementation**:
  - Require identity verification before account changes
  - Provide scripts and escalation paths for suspicious requests
  - Run regular simulated phishing exercises

### Testing and Detection

### [ ] Run Regular Penetration Tests and Retests
- **Description**: Independent testing finds issues that automated tools miss.
- **Severity**: HIGH
- **Implementation**:
  - Test at least yearly and after major releases
  - Retest every critical and high finding after the fix
  - Track findings to closure in the same backlog as other work

### [ ] Fuzz Parsers and File Handlers
- **Description**: Parsers for files, images, and structured input are frequent sources of crashes and memory bugs.
- **Severity**: MEDIUM
- **Implementation**:
  - Fuzz parsers in CI or on a schedule (for example with OSS-Fuzz or libFuzzer)
  - Add regression tests for every crash found

### [ ] Tune Security Detections and Reduce Alert Fatigue
- **Description**: Alerts nobody acts on hide real incidents.
- **Severity**: HIGH
- **OWASP**: 09 - Logging and Monitoring Failures
- **Implementation**:
  - Write detections for known attacker techniques and review them quarterly
  - Track alert volume and the share of alerts that lead to action
  - Retire or tune noisy rules

### [ ] Maintain On-Call and Security Runbooks
- **Description**: Responders need clear steps when an alert fires at night.
- **Severity**: HIGH
- **Implementation**:
  - Keep an on-call rotation with escalation paths
  - Write a runbook for each high-severity alert type
  - Practice runbooks in tabletop exercises

### Business Abuse

### [ ] Detect and Limit Fraud, Bot Accounts, and Account Farming
- **Description**: Automated accounts and fraudulent sign-ups distort metrics and cost money.
- **Severity**: HIGH
- **Implementation**:
  - Use device signals, velocity checks, and CAPTCHA on sign-up
  - Verify email or phone for new accounts
  - Review and suspend clusters of linked accounts

### [ ] Prevent Promo, Referral, and Credit Abuse
- **Description**: Coupons, referral rewards, and credits can be farmed or reused.
- **Severity**: MEDIUM
- **Implementation**:
  - Limit each reward to one per verified identity or payment method
  - Make credit redemption atomic (see the race-condition item)
  - Monitor redemption patterns and set caps

### [ ] Detect and Limit Scraping and Content Theft
- **Description**: Bulk scraping can copy your content, overload your systems, and reveal business data.
- **Severity**: MEDIUM
- **Implementation**:
  - Rate-limit by account and IP, and detect automated browsing patterns
  - Require login for bulk data access
  - Watermark or monitor high-value content where practical

---

## Phase 4: AI Red Team (Ongoing)

- Run automated scans (garak, PyRIT, promptfoo) on every release
- Re-run the jailbreak and injection regression suite after every model or prompt change
- Hold a manual AI red-team exercise each quarter, covering agents, RAG, and MCP tools
- Map findings to MITRE ATLAS techniques and track them in the same backlog as other security issues
