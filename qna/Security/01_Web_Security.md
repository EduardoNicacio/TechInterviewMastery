# Web Security – Interview Questions & Answers

**Question**: How can you prevent SQL injection in application code?

**Answer**: Use parameterized queries or prepared statements so user input is treated as data, not executable SQL. ORMs like Entity Framework, Hibernate, and Django ORM handle this automatically when using LINQ or query builders. Never concatenate raw user input into SQL strings.

```csharp
// C# with Dapper
var user = await connection.QueryFirstOrDefaultAsync<User>(
    "SELECT * FROM Users WHERE Id = @Id", new { Id = userId });
```

```java
// Java with JDBC
PreparedStatement stmt = conn.prepareStatement("SELECT * FROM Users WHERE email = ?");
stmt.setString(1, userInput);
```

```python
# Python with psycopg2
cur.execute("SELECT * FROM users WHERE email = %s", (user_input,))
```

---

**Question**: What are the three types of Cross-Site Scripting (XSS) and how do you mitigate each?

**Answer**: Reflected XSS (input in URL/response), Stored XSS (persisted in database), and DOM-based XSS (client-side script manipulation). Mitigate with context-aware output encoding, a strict Content-Security-Policy header, and input sanitization. Avoid using `innerHTML` or `dangerouslySetInnerHTML`.

```javascript
// Safe DOM manipulation
const userText = document.createTextNode(userInput);
document.getElementById('message').appendChild(userText);

// React (avoids injection)
<div>{userInput}</div>
```

```csharp
// ASP.NET Core Razor auto-encodes
@Model.UserInput
```

---

**Question**: How does a CSRF attack work and what defenses are effective?

**Answer**: An attacker tricks an authenticated user into submitting a malicious request to a trusted site by embedding a form or image in another page. Defend with anti-CSRF tokens validated server-side, `SameSite=Strict/Lax` cookies, and custom request headers (e.g., `X-Requested-With`). Never rely solely on cookies for authentication without CSRF protection.

```csharp
// ASP.NET Core – built-in anti-forgery
[ValidateAntiForgeryToken]
public IActionResult UpdateProfile(ProfileModel model) { ... }

// Razor view
<form asp-action="UpdateProfile" method="post">
    @Html.AntiForgeryToken()
    ...
</form>
```

```javascript
// Express with csurf (deprecated, use csrf-csrf)
import { doubleCsrf } from "csrf-csrf";
const { generateToken, doubleCsrfProtection } = doubleCsrf();
app.post("/api/transfer", doubleCsrfProtection, handler);
```

---

**Question**: What is SSRF and how can you prevent it?

**Answer**: Server-Side Request Forgery occurs when an attacker makes the server fetch internal resources by manipulating a URL parameter. Use an allowlist of permitted schemes and hostnames, validate and sanitize all URL inputs, and avoid passing raw URLs to fetch functions. Implement network segmentation so application servers cannot reach internal metadata endpoints (e.g., 169.254.169.254).

```python
from urllib.parse import urlparse

ALLOWED_HOSTS = {"api.example.com", "trusted-cdn.com"}
def safe_fetch(url: str):
    parsed = urlparse(url)
    if parsed.hostname not in ALLOWED_HOSTS or parsed.scheme not in {"https"}:
        raise ValueError("Blocked URL")
```

```java
// Java – validate before connecting
URI uri = new URI(userInput);
if (!"https".equals(uri.getScheme()) || !allowedHosts.contains(uri.getHost())) {
    throw new SecurityException("URL not allowed");
}
```

---

**Question**: What is an Insecure Direct Object Reference (IDOR) vulnerability?

**Answer**: IDOR occurs when an application exposes direct references to internal objects (e.g., `GET /order/123`) without verifying the requester's authorization to access that specific object. Fix by enforcing ownership or permission checks on every access — never trust that a user can access an object just because they know its identifier.

```csharp
// Vulnerable
public IActionResult GetOrder(int orderId)
    => Ok(_db.Orders.Find(orderId));

// Fixed – check ownership
public IActionResult GetOrder(int orderId)
{
    var order = _db.Orders.FirstOrDefault(o => o.Id == orderId && o.UserId == currentUserId);
    if (order == null) return NotFound();
    return Ok(order);
}
```

---

**Question**: How can you prevent path traversal attacks in file-serving endpoints?

**Answer**: Validate that the resolved path stays within an intended root directory by using `Path.GetFullPath` and checking the result starts with the allowed base path. Reject inputs containing `..`, null bytes, or absolute paths. Serve files through a controlled API rather than directly exposing the filesystem.

```python
import os
from pathlib import Path

BASE_DIR = Path("/app/uploads/")
def safe_read(filename: str):
    full_path = (BASE_DIR / filename).resolve()
    if not full_path.is_relative_to(BASE_DIR):
        raise PermissionError("Path traversal detected")
    return full_path.read_bytes()
```

---

**Question**: What are the risks of insecure deserialization and how do you mitigate them?

**Answer**: Untrusted data deserialized by libraries like Java's `ObjectInputStream`, Python's `pickle`, or .NET's `BinaryFormatter` can execute arbitrary code or cause denial-of-service. Use safe, structured data formats (JSON over binary serialization), apply integrity checks (HMAC/signing), or use allowlisted deserialization. Prefer `System.Text.Json` over `BinaryFormatter` in .NET, and use `defusedxml` or `json` in Python.

```java
// Java – use allowlist with ValidatingObjectInputStream
ValidatingObjectInputStream ois = new ValidatingObjectInputStream(input);
ois.accept(User.class, Order.class);
Object obj = ois.readObject();
```

```python
# Python – avoid pickle for untrusted data; use JSON
import json
data = json.loads(user_input)
```

---

**Question**: What is an XXE attack and how do you defend against it?

**Answer**: XML External Entity injection exploits XML parsers that process external entities, potentially exposing internal files, performing SSRF, or causing denial-of-service. Disable DTD processing entirely in your XML parser, or use a safer format like JSON. Most modern parsers have DTD disabled by default, but legacy configurations may still be vulnerable.

```python
# Python with defusedxml (safe by default)
from defusedxml import ElementTree as ET
root = ET.fromstring(xml_string)
```

```csharp
// .NET – disable DTD
var settings = new XmlReaderSettings { DtdProcessing = DtdProcessing.Prohibit };
```

---

**Question**: What constitutes a security misconfiguration and how do you prevent it?

**Answer**: Common misconfigurations include default credentials, verbose error messages exposing stack traces, unsecured cloud storage, unnecessary open ports, and missing security headers. Prevent by hardening default templates, minimizing attack surface (disable unused features), automating infrastructure-as-code reviews, and running regular configuration audits with tools like Scuba or CIS benchmarks.

```javascript
// express.js security checklist
app.use(helmet());
app.disable('x-powered-by');
app.set('env', 'production');
```

---

**Question**: How should you manage third-party component vulnerabilities?

**Answer**: Maintain a Software Bill of Materials (SBOM) for all dependencies, run automated vulnerability scanning (Dependabot, Snyk, Trivy), and subscribe to CVE feeds. Pin dependency versions, review transitive dependencies, and automate patching with CI/CD gates that block builds with critical vulnerabilities.

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
```

---

**Question**: Describe the OAuth 2.0 Authorization Code flow with PKCE.

**Answer**: The client requests an authorization code from the authorization server, which the user approves. The client exchanges this code (along with the original `code_verifier` via PKCE) for an access token at the token endpoint. PKCE prevents interception attacks by binding the authorization code to the client's unique challenge; it is mandatory for public clients (SPA/mobile).

```
// Authorization request (with PKCE)
GET /authorize?response_type=code&client_id=myapp
    &code_challenge_method=S256
    &code_challenge=<base64url(sha256(code_verifier))>
    &redirect_uri=...&state=abc

// Token request
POST /token
grant_type=authorization_code
&code=<auth_code>&code_verifier=<original_verifier>
&redirect_uri=...&client_id=myapp
```

```javascript
// JavaScript – PKCE generation (using Web Crypto API)
const verifier = crypto.randomUUID() + crypto.randomUUID();
const challenge = btoa(String.fromCharCode(...new Uint8Array(
    await crypto.subtle.digest('SHA-256', new TextEncoder().encode(verifier))
  ))).replace(/\+/g, '-').replace(/\//g, '_').replace(/=+$/, '');
```

---

**Question**: How does OpenID Connect differ from plain OAuth 2.0?

**Answer**: OpenID Connect (OIDC) adds an identity layer on top of OAuth 2.0 by returning an ID token (a JWT) alongside the access token. The ID token contains user claims (sub, name, email) and is digitally signed, enabling the client to authenticate the user. OAuth 2.0 alone only authorizes access to resources without providing identity information.

```csharp
// Simplified — real code must validate signature, issuer, audience, and expiry via JWKS
var handler = new JwtSecurityTokenHandler();
var idToken = handler.ReadJwtToken(idTokenString);
var userId = idToken.Subject;
```

---

**Question**: Explain the structure of a JWT and the difference between RS256 and HS256.

**Answer**: A JWT consists of three Base64URL-encoded segments (header, payload, signature) separated by dots. HS256 uses a symmetric HMAC with a single shared secret — faster but requires the secret to be distributed, making it unsuitable for multi-service environments. RS256 uses asymmetric RSA keys: the issuer signs with a private key and any service can verify with the public key, which is safer for distributed systems.

```
// JWT structure
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIn0.
s9kLQWJkfQ...

Algorithm        Key type       Verification        Use case
HS256 (HMAC)     Symmetric      Same secret         Single service
RS256 (RSA)      Asymmetric     Public JWKS URL     Microservices / multi-party
ES256 (ECDSA)    Asymmetric     Public JWKS URL     Smaller tokens, faster
```

---

**Question**: What best practices should you follow for JWT validation and lifecycle?

**Answer**: Always validate the signature, issuer (`iss`), audience (`aud`), expiration (`exp`), and not-before (`nbf`) claims. Use short expiration times (15-30 minutes), implement token rotation, and maintain a deny-list for revoked tokens. Store JWTs in memory (not localStorage) for SPAs to prevent XSS exfiltration.

```python
import jwt
from jwt import PyJWKClient

jwks_client = PyJWKClient("https://auth.example.com/.well-known/jwks.json")
signing_key = jwks_client.get_signing_key_from_jwt(token)

payload = jwt.decode(token, signing_key.key, algorithms=["RS256"],
                     audience="myapp", issuer="https://auth.example.com")
```

---

**Question**: How would you design a secure session management system?

**Answer**: Use server-side opaque session identifiers (cryptographically random tokens), stored in `Secure`, `HttpOnly`, `SameSite=Strict` cookies. Never expose the session token in URLs or logs. Implement absolute and sliding expiration, rotation after privilege escalation, and server-side revocation.

```csharp
// ASP.NET Core – secure session configuration
builder.Services.AddSession(options =>
{
    options.Cookie.HttpOnly = true;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.SameSite = SameSiteMode.Strict;
    options.IdleTimeout = TimeSpan.FromMinutes(20);
});
```

---

**Question**: Why is multi-factor authentication (MFA) important and what factors are commonly used?

**Answer**: MFA requires two or more independent credentials (something you know, have, and are), reducing the impact of password compromise. Common factors include TOTP authenticator apps, hardware security keys (WebAuthn/FIDO2), SMS codes, and biometrics. Prefer phishing-resistant methods like WebAuthn over SMS where possible.

```
// TOTP generation principle (RFC 6238)
TOTP = HMAC-SHA1(secret, floor(current_time / 30)) truncated to 6 digits
```

```javascript
// WebAuthn registration example (simplified)
const credential = await navigator.credentials.create({
    publicKey: {
        challenge: new Uint8Array(32),
        rp: { name: "MyApp" },
        user: { id: userId, name: "user@example.com", displayName: "User" },
        pubKeyCredParams: [{ type: "public-key", alg: -7 }] // ES256
    }
});
```

---

**Question**: What is SAML 2.0 and how does it differ from OIDC?

**Answer**: SAML 2.0 is an XML-based SSO protocol using assertions from an Identity Provider to a Service Provider. It is heavier than OIDC, relies on XML signatures, and is common in enterprise environments (ADFS, Okta). OIDC is JSON-based, simpler to implement, and more suited to modern web/mobile apps. SAML uses browser redirects with XML payloads; OIDC uses JWTs and REST.

---

**Question**: Compare RBAC, ABAC, and ReBAC. When would you use each?

**Answer**: RBAC assigns permissions via roles (least flexible, best for simple hierarchies). ABAC evaluates policies against user/resource/context attributes (most flexible, best for complex multi-tenant systems). ReBAC (Relationship-Based Access Control) derives permissions from a graph of relationships between entities (best for social or collaborative platforms). Follow the principle of least privilege regardless: grant only the minimum permissions needed.

```
# RBAC: role → permissions
role "editor" { permissions = ["read", "write"] }

# ABAC: attribute-based policy
"Effect": "Allow",
"Condition": {
    "StringEquals": { "aws:PrincipalTag/department": "engineering" }
}
```

---

**Question**: What is the difference between whitelist and blacklist input validation?

**Answer**: Whitelisting (allowlist) defines explicitly permitted values and patterns and rejects everything else — it is far more secure. Blacklisting tries to block known malicious patterns but is easily bypassed by novel attacks. Always normalize inputs (canonicalize Unicode, trim whitespace) before validation to prevent encoding-based bypasses.

```csharp
// Whitelist approach
var allowedCountries = new HashSet<string> { "US", "CA", "UK" };
if (!allowedCountries.Contains(input?.ToUpperInvariant()))
    throw new ValidationException("Invalid country");
```

```javascript
// Regex allowlist for a username
const usernameRegex = /^[a-zA-Z0-9_]{3,20}$/;
if (!usernameRegex.test(input)) return res.status(400).send('Invalid username');
```

---

**Question**: How would you implement rate limiting for a production API?

**Answer**: Use a sliding window or token bucket algorithm keyed by user ID or IP address. Apply different limits per endpoint (e.g., stricter for auth endpoints). Return `429 Too Many Requests` with `Retry-After` header. Use a fast distributed store like Redis for shared counters across instances.

```csharp
// ASP.NET Core with AspNetCoreRateLimit
builder.Services.Configure<IpRateLimitOptions>(options =>
{
    options.GeneralRules = new List<RateLimitRule>
    {
        new() { Endpoint = "*/api/*", Limit = 100, Period = "1m" },
        new() { Endpoint = "*/auth/login", Limit = 5, Period = "1m" }
    };
});
```

```python
# FastAPI with slowapi
from slowapi import Limiter, _rate_limit_exceeded_handler
limiter = Limiter(key_func=lambda request: request.client.host)
app.state.limiter = limiter

@app.post("/api/login")
@limiter.limit("5/minute")
async def login(request: Request): ...
```

---

**Question**: How does CORS work and what security considerations should you keep in mind?

**Answer**: CORS uses HTTP headers to tell browsers whether a web app at one origin can access resources from another. The server must validate the `Origin` header against an allowlist and respond with `Access-Control-Allow-Origin`. Preflight `OPTIONS` requests are sent for non-simple requests. Never use `Access-Control-Allow-Origin: *` with credentials, and avoid reflecting arbitrary origins.

```csharp
// ASP.NET Core – restrictive CORS policy
builder.Services.AddCors(options =>
{
    options.AddPolicy("Restricted", p =>
        p.WithOrigins("https://myapp.com")
         .AllowMethods("GET", "POST")
         .AllowCredentials());
});
```

```javascript
// Express
app.use(cors({
    origin: ['https://myapp.com', 'https://admin.myapp.com'],
    credentials: true,
    methods: ['GET', 'POST']
}));
```

---

**Question**: Which HTTP security headers and CSP directives should you set on every response?

**Answer**: Set `Strict-Transport-Security` (HSTS) with `max-age=31536000; includeSubDomains`, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, and `Referrer-Policy: strict-origin-when-cross-origin`. For CSP, use `default-src 'self'`, `script-src` with nonces/hashes (avoid `'unsafe-inline'`), `object-src 'none'`, and `frame-ancestors 'none'`. These prevent clickjacking, MIME sniffing, XSS, and information leakage.

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), camera=(), microphone=()
Content-Security-Policy: default-src 'self';
    script-src 'strict-dynamic' 'nonce-abc123' https://js.stripe.com;
    object-src 'none'; base-uri 'none'; frame-ancestors 'none';
    report-uri /csp-report;
```

---

**Question**: How do you manage secrets in a production system?

**Answer**: Use a dedicated secrets manager (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault) with automatic rotation, access audit logs, and encryption at rest. Never store secrets in code or config files. Secrets should be injected at runtime via a secrets manager, environment variables from a trusted source (e.g., Vault, cloud provider secret stores), or sidecar containers, not baked into images or repositories.

```bash
# Vault CLI – inject secrets at runtime
vault kv get -field=password secret/myapp/db
```

```csharp
// .NET – injected via configuration
builder.Configuration.AddAzureKeyVault(
    new Uri("https://myvault.vault.azure.net/"),
    new DefaultAzureCredential());
```

---

**Question**: What hashing algorithms should you use for passwords and why?

**Answer**: Use adaptive, salted hashing algorithms like bcrypt, scrypt, or Argon2id. Unlike SHA-256 (which is fast and designed for integrity, not password storage), these include a configurable work factor that makes brute-force attacks infeasible. Argon2id is the current OWASP-recommended choice.

```python
import argon2
ph = argon2.PasswordHasher(time_cost=3, memory_cost=65536, parallelism=4)
hash = ph.hash("user_password")

# Verification
try:
    ph.verify(hash, "user_password")
except argon2.exceptions.VerifyMismatchError:
    # wrong password
    pass
```

```csharp
// .NET with BCrypt
var hash = BCrypt.Net.BCrypt.HashPassword(password, workFactor: 12);
bool valid = BCrypt.Net.BCrypt.Verify(password, hash);
```

---

**Question**: What is the difference between encryption at rest and in transit?

**Answer**: Encryption at rest protects stored data (AES-256-GCM for files/databases) using keys managed by services like AWS KMS or Azure Key Vault. Encryption in transit (TLS 1.3) protects data during network transfer. Both are essential: at rest defends against storage breaches, while in transit prevents interception (MITM) attacks. Never use TLS 1.0/1.1 or weak cipher suites.

```bash
# TLS 1.3 only configuration (Nginx)
ssl_protocols TLSv1.3;
ssl_ciphers TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256;
```

---

**Question**: Compare RSA and ECDSA for digital signatures.

**Answer**: ECDSA offers equivalent security to RSA with smaller key sizes (e.g., 256-bit ECC ≈ 3072-bit RSA). ECDSA is faster at key generation and signing, while RSA is faster at verification. For JWT signing, ECDSA (ES256) is increasingly preferred over RSA (RS256) due to smaller token sizes and better performance.

```
Security comparison:
RSA-2048    → ~112-bit security level
RSA-3072    → ~128-bit security level
ECDSA P-256 → ~128-bit security level (smaller key, faster signing)

Key sizes:
RSA-2048:    2048 bits
ECDSA P-256:  256 bits
```

---

**Question**: How does asymmetric encryption and digital signatures work?

**Answer**: Asymmetric encryption uses a key pair: the public key encrypts, and the private key decrypts. Digital signatures reverse this — the private key signs a hash of the message, and the public key verifies it. This enables non-repudiation (the signer cannot deny signing) and secure key exchange without sharing secrets.

```python
# Signing with RSA
from cryptography.hazmat.primitives import hashes, asymmetric
signature = private_key.sign(data, asymmetric.padding.PKCS1v15(), hashes.SHA256())

# Verification
public_key.verify(signature, data, asymmetric.padding.PKCS1v15(), hashes.SHA256())
```

---

**Question**: Describe the TLS handshake and differences between TLS 1.2 and 1.3.

**Answer**: In TLS 1.2, the handshake requires two round trips (ClientHello → ServerHello + Certificate → ClientKeyExchange → ChangeCipherSpec → Finished). TLS 1.3 reduces this to one round trip (1-RTT) by combining key exchange and negotiation, removes weak cipher suites, and uses forward-secrecy-only key exchange (ECDHE). TLS 1.3 also provides 0-RTT for resumption (with replay protection caveats).

```
TLS 1.2 handshake (2 RTT):
ClientHello → ServerHello + Certificate + ServerHelloDone
ClientKeyExchange + ChangeCipherSpec → Finished
ChangeCipherSpec + Finished

TLS 1.3 handshake (1 RTT):
ClientHello (key_share) → ServerHello + Certificate + CertificateVerify + Finished
Client Finish
```

---

**Question**: What does it mean to "fail securely"?

**Answer**: Failing securely means that when an error occurs, the system defaults to the most restrictive state — denying access rather than granting it. For example, if an authorization check throws an exception, the request should be rejected. Error messages should be generic ("Access denied") without revealing whether the user or resource was the issue.

```csharp
// Bad – fails open
try { var user = await FindUser(id); return Ok(user); }
catch { return Ok(null); }

// Good – fails closed
try {
    var user = await FindUser(id);
    return user != null ? Ok(user) : NotFound();
}
catch { return StatusCode(500); }
```

---

**Question**: Explain the defense-in-depth principle with examples.

**Answer**: Defense in depth layers multiple independent security controls so that if one fails, others still protect the system. Example layers: WAF → HTTPS → CSRF tokens → input validation → parameterized queries → least-privilege DB user → encrypted storage. An attacker must breach every layer to succeed.

---

**Question**: How do you secure the software supply chain?

**Answer**: Generate SBOMs for all dependencies, scan with tools like Dependabot, Snyk, or Trivy in CI/CD, and enforce signed commits. Pin dependency hashes (integrity checks in package-lock.json), use artifact registries proxying upstream sources, and have a vulnerability response playbook. Sign your releases with cosign or GPG.

```bash
# Generate SBOM with Syft
syft packages . -o cyclonedx-json > sbom.json

# Scan with Trivy
trivy image myapp:latest --severity CRITICAL,HIGH

# Verify signed commits
git verify-commit HEAD
```

---

**Question**: Compare SAST, DAST, and IAST.

**Answer**: SAST (Static) scans source code without executing it, catching issues early. DAST (Dynamic) tests the running application from the outside, finding runtime vulnerabilities. IAST (Interactive) instruments the application to monitor code paths during normal use, combining SAST's coverage with DAST's runtime context with fewer false positives.

| Tool  | Phase       | Approach           | False Positives |
|-------|-------------|--------------------|-----------------|
| SAST  | Build/IDE   | Source code scan   | Higher          |
| DAST  | QA/Staging  | Black-box attack   | Lower           |
| IAST  | Testing     | Instrumented app   | Lowest          |

---

**Question**: What should a security code review checklist include?

**Answer**: Authentication and authorization checks on every endpoint, input validation (whitelist + normalization), output encoding context, proper cryptography usage (no custom algorithms, correct key sizes), error handling (no stack traces), session management (HttpOnly/Secure cookies), dependency reviews, and secrets (none in code or configs).

---

**Question**: How do AWS IAM policies work and how do Security Groups differ from NACLs?

**Answer**: IAM policies define permissions via a JSON document with `Effect`, `Action`, `Resource`, and optional `Condition` — granting or denying specific API actions. Security Groups are stateful instance-level firewalls (allow rules only), while NACLs are stateless subnet-level filters supporting both allow and deny rules. Use IAM for API access control and Security Groups/NACLs for network access.

```json
{
    "Effect": "Allow",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::my-bucket/*",
    "Condition": {
        "IpAddress": { "aws:SourceIp": "10.0.0.0/16" }
    }
}
```

---

**Question**: How would you manage secrets rotation and encryption keys in the cloud?

**Answer**: Use a dedicated secrets manager (AWS Secrets Manager, Azure Key Vault, HashiCorp Vault) with automatic rotation schedules. Encrypt data at rest using a KMS key hierarchy: a Customer Master Key (CMK) encrypts Data Encryption Keys (DEKs), which encrypt the actual data. Rotate CMKs annually (or on compromise) and DEKs more frequently. Use envelope encryption to store the encrypted DEK alongside the data.

```
KMS Key Hierarchy:
CMK (Root Key) → encrypts → DEK (Data Encryption Key) → encrypts → actual data
                          → stored alongside data (envelope encryption)

Automation: AWS Secrets Manager rotates RDS credentials every 30 days.
```
