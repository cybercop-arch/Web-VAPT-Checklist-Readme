# Web-VAPT-Checklist-Readme

WEB VAPT CHECKLIST

PHASE 1 — RECON
Fingerprinting & Discovery

 Find all subdomains using subfinder, amass, or crt.sh
 Identify tech stack — server, CMS, frameworks using Wappalyzer or whatweb
 Check HTTP response headers for software version leaks
 Google dork the target: site:target.com filetype:pdf inurl:admin etc.
 Check robots.txt and sitemap.xml for hidden or sensitive paths
 Use Wayback Machine to find old or deleted endpoints

JS File Analysis

 Extract hidden endpoints and secrets from .js files using linkfinder or jsbeautifier
 Look for hardcoded API keys, tokens, or credentials inside JS files
 Identify API routes not visible anywhere in the UI

Initial Scanning

 Run nikto for banner grabbing and common misconfigs
 Directory brute force — /admin, /backup, /config, /uploads using ffuf or gobuster
 Enumerate API documentation pages — /swagger, /api-docs, /openapi.json


PHASE 2 — AUTHENTICATION & SESSION
Login & Credentials

 Test default credentials on admin panels — admin/admin, admin/password
 Brute force login and check if account lockout kicks in (Burp Intruder, hydra)
 Account enumeration — look for different error messages for valid vs invalid username
 Password reset token reuse — request two resets and check if both tokens still work

Session Management

 Check session token entropy — are the tokens short, sequential, or guessable?
 Session fixation — inject a known session ID before login and see if it stays
 Replay an old token after logout — is the session properly invalidated?
 Check cookie flags — Secure, HttpOnly, and SameSite should all be set

JWT & MFA

 JWT alg:none attack — strip the signature and set algorithm to none
 JWT weak secret — brute force the HS256 secret using hashcat or jwt_tool
 JWT key confusion — sign with HS256 using the RS256 public key
 MFA bypass — manipulate the response or try skipping the MFA step entirely


PHASE 3 — INJECTION
SQL Injection

 Manual SQLi — test all inputs with ' OR 1=1-- and error-based payloads
 Automated SQLi — run sqlmap across all URL parameters and form fields
 Blind SQLi — time-based: ' OR SLEEP(5)-- to confirm without visible error
 Out-of-band SQLi — DNS or HTTP callback if time-based is blocked

Other Injection Types

 Command injection — try ; id, | whoami, && ls in any OS-interacting inputs
 SSTI — test {{77}}, ${77}, <%= 7*7 %> in template-rendered fields
 NoSQL injection — try {"$gt":""} in JSON body parameters
 XXE — inject an external XML entity in XML body or file upload
 LDAP injection — try *)( or )(objectClass= in directory-based inputs


PHASE 4 — BROKEN ACCESS CONTROL
IDOR & Object-Level Access

 IDOR — change object IDs in API calls: /api/user/123 → /api/user/124
 Horizontal privilege escalation — access another user's data using your session
 Vertical privilege escalation — access admin-only endpoints as a normal user
 Mass assignment — send extra JSON fields like role:admin or isAdmin:true

URL & Workflow Access

 Force browsing — directly visit /admin, /dashboard, /config without logging in
 Path traversal — try ../../etc/passwd in any file name or download parameter
 API method abuse — test DELETE, PUT, PATCH on restricted endpoints
 Multi-step workflow skip — jump from step 1 directly to step 3


PHASE 5 — XSS & CLIENT-SIDE
XSS Types

 Reflected XSS — inject <script>alert(1)</script> in URL params and form fields
 Stored XSS — submit payload in profile fields, comments, or usernames
 DOM XSS — check JS sinks like document.write(location.hash) for injection points
 XSS via file upload — upload an SVG file with an embedded script tag
 HTML injection — inject <b>test</b> in email or order confirmation fields

CSP, Framing & CSRF

 Check the CSP header — is it missing, too permissive, or bypassable?
 Clickjacking — is X-Frame-Options or CSP frame-ancestors missing?
 Try embedding the target in an iframe from your own controlled page
 CSRF — remove the CSRF token from a state-changing request and resend it
 Check SameSite cookie attribute — missing SameSite is a CSRF risk


PHASE 6 — FILE HANDLING
File Upload Attacks

 Upload a PHP or ASPX shell with a renamed extension like shell.php.jpg
 MIME type bypass — change Content-Type to image/jpeg while sending a malicious file
 Polyglot file — a valid image that is simultaneously valid PHP
 Zip slip — upload a zip with ../../malicious.php inside the path
 Check if uploaded files land in web root and can be executed

File Download Attacks

 Path traversal in download endpoint — ?file=../../etc/passwd
 SSRF via file URL — make the server fetch internal resources on your behalf


PHASE 7 — API SECURITY
API Enumeration

 Find all API endpoints from JS files, Swagger docs, and Burp sitemap
 Test unauthenticated access to every API endpoint you find
 Check if the API returns excess fields not shown in the UI

API Attacks

 BOLA — swap user IDs in API paths to pull another user's objects
 Mass assignment — inject extra fields like role or admin in POST/PUT body
 Rate limiting — flood API endpoints repeatedly and see if throttling kicks in
 GraphQL introspection — send {__schema{types{name}}} to map the full schema
 GraphQL batching attack — send many queries in a single request to bypass limits


PHASE 8 — MISCONFIGURATIONS
Server & Transport

 TLS version check — SSLv3 and TLS 1.0/1.1 should be disabled (use testssl.sh)
 Check for weak cipher suites — RC4, DES, and EXPORT ciphers are red flags
 HSTS header — is it missing or not preloaded?
 HTTP methods — is TRACE, PUT, or DELETE allowed where it shouldn't be?

Exposed Resources

 Directory listing — try /images/, /uploads/, /backup/ and see if files are listed
 Debug pages exposed — /phpinfo.php, /test, /info, /.env accessible?
 Cloud bucket enumeration — public S3, GCP, or Azure blob (grayhatwarfare, lazys3)
 CORS misconfiguration — set Origin: evil.com and check Access-Control-Allow-Origin
 SMTP header injection — inject newlines into contact form email fields


PHASE 9 — BUSINESS LOGIC
Logic Flaws

 Negative quantity or price manipulation in cart or checkout flow
 Coupon code reuse — apply the same discount code multiple times
 Skip payment step — go directly to /order-confirm after adding to cart
 Race condition — send parallel requests on limited stock or balance transfers
 Workflow bypass — manipulate hidden fields or params to skip business rules
