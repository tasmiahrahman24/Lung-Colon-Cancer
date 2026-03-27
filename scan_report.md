# 🔒 Security Scan Report

**Generated:** 2026-03-26 23:32 UTC  
**Scan Root:** `/Users/tasmiahrahman/Documents/New Engine/HIPAA Demo App/hipaachecker.health-feature-docker-back-end`  
**Frameworks:** hipaa, soc2

## Summary

| Metric | Count |
|--------|-------|
| Raw Findings | 101 |
| Compliance Violations | 167 |
| Attack Paths | 0 |
| Systemic Patterns | 0 |

### Severity Breakdown

- 🔴 **CRITICAL**: 59
- 🟠 **HIGH**: 92
- 🟡 **MEDIUM**: 16

## 🚨 Critical & High Violations

### [HIGH] Access Control — Access Control Exception
- **Framework:** hipaa
- **Rule:** HIPAA-AC-004 — Access Control
- **File:** `app/controllers/health_controller.rb`
- **Description:** An unauthenticated health check endpoint exposes internal environment, database, and Redis status information without access restrictions. This constitutes an access control exception by allowing unauthorized users to obtain sensitive system context that could facilitate attacks against systems handling ePHI.
- **Remediation:** Restrict the health check endpoint to authenticated and authorized users only, or limit exposure to minimal status (e.g., HTTP 200). Remove environment and backend service details from responses or protect the endpoint behind network-level controls.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Security
- **File:** `app/controllers/health_controller.rb`
- **Description:** Publicly accessible system health details indicate insufficient authorization controls over sensitive operational information, which undermines SOC 2 access control expectations.
- **Remediation:** Implement authorization checks for diagnostic endpoints and ensure only authorized roles or internal monitoring systems can access operational details.

### [HIGH] Audit Controls — User Event Tracking
- **Framework:** hipaa
- **Rule:** HIPAA-AUD-002 — Audit Controls
- **File:** `app/models/hipaa_risk_score_calculator.rb`
- **Description:** Logging raw exception messages using unstructured output (`puts`) can expose internal system details in logs without proper controls, undermining audit logging discipline and increasing the risk of inappropriate disclosure.
- **Remediation:** Replace `puts` with a structured logging framework. Sanitize exception messages, avoid logging sensitive context, and ensure logs are centrally managed with access controls and retention policies.

### [HIGH] Audit Logging and Monitoring
- **Framework:** soc2
- **Rule:** SOC2-SEC-005 — Security
- **File:** `app/models/hipaa_risk_score_calculator.rb`
- **Description:** Uncontrolled logging of exception details does not meet SOC 2 expectations for secure, consistent, and monitored audit logging.
- **Remediation:** Implement centralized, structured logging with log levels, redaction, and monitoring alerts. Ensure access to logs is restricted and reviewed.

### [HIGH] Transmission Security — Trusted Server Communication
- **Framework:** hipaa
- **Rule:** HIPAA-NET-001 — Transmission Security
- **File:** `config/initializers/content_security_policy.rb`
- **Description:** The absence of an enforced Content Security Policy increases the risk of XSS and data exfiltration attacks, which could compromise the secure transmission and handling of ePHI in client-server interactions.
- **Remediation:** Enable and enforce a restrictive Content Security Policy that limits script, style, image, and connection sources to trusted origins only.

### [HIGH] Secure Data Transmission
- **Framework:** soc2
- **Rule:** SOC2-SEC-004 — Security
- **File:** `config/initializers/content_security_policy.rb`
- **Description:** Lack of CSP weakens browser-level protections against malicious content that could intercept or exfiltrate sensitive data, conflicting with SOC 2 secure transmission principles.
- **Remediation:** Configure and deploy CSP headers appropriate to the application’s threat model and regularly review them for effectiveness.

### [HIGH] Transmission Security — Trusted Server Communication
- **Framework:** hipaa
- **Rule:** HIPAA-NET-001 — Transmission Security
- **File:** `config/initializers/permissions_policy.rb`
- **Description:** Without a Permissions-Policy header, browser features may be abused by malicious scripts, increasing the attack surface for exfiltration of sensitive data in applications handling ePHI.
- **Remediation:** Enable and configure a restrictive Permissions-Policy header to disable unnecessary browser features such as camera, microphone, and USB access.

### [HIGH] Encryption at Rest and in Transit
- **Framework:** soc2
- **Rule:** SOC2-SEC-001 — Security
- **File:** `config/initializers/permissions_policy.rb`
- **Description:** While not a direct encryption failure, missing client-side security headers weaken the overall security posture expected under SOC 2 controls for protecting sensitive data.
- **Remediation:** Implement Permissions-Policy headers as part of a defense-in-depth strategy alongside encryption and secure transmission controls.

### [CRITICAL] PHI Database Encryption
- **Framework:** hipaa
- **Rule:** HIPAA-PHI-001 — Encryption and Decryption
- **File:** `patterns/released/express/phi_encryption.yaml`
- **Description:** Non-functional security detection rules undermine safeguards intended to ensure encryption and protection of ePHI, creating a risk that unencrypted PHI storage or transmission goes undetected.
- **Remediation:** Implement valid detection patterns for encryption, transmission security, and data storage rules. Regularly test rule effectiveness to ensure safeguards operate as intended.

### [CRITICAL] Encryption at Rest and in Transit
- **Framework:** soc2
- **Rule:** SOC2-SEC-001 — Security
- **File:** `patterns/released/express/phi_encryption.yaml`
- **Description:** Ineffective security controls provide a false sense of encryption assurance, conflicting with SOC 2 requirements for protecting sensitive data at rest and in transit.
- **Remediation:** Define and validate actual security controls and monitoring logic that verify encryption and secure transmission are properly implemented.

### [HIGH] Integrity — Input Validation
- **Framework:** hipaa
- **Rule:** HIPAA-INT-001 — Integrity
- **File:** `patterns/released/express/phi_encryption.yaml`
- **Description:** Misconfigured and inconsistent rule identifiers undermine the integrity of security analysis, potentially allowing encryption and transmission issues affecting ePHI to go undetected.
- **Remediation:** Correct rule IDs and references to ensure consistency. Validate rule dependencies and perform integrity checks on configuration files.

### [CRITICAL] Secure Data Storage
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-010 — Encryption and Decryption
- **File:** `patterns/released/express_cvss/phi_encryption.yaml`
- **Description:** Non-functional encryption and transmission detection rules, combined with misconfigurations, undermine required safeguards for secure storage and handling of ePHI.
- **Remediation:** Implement functional detection logic, correct rule definitions, and validate that encryption and secure transmission requirements are actively enforced.

### [CRITICAL] Encryption at Rest and in Transit
- **Framework:** soc2
- **Rule:** SOC2-SEC-001 — Security
- **File:** `patterns/released/express_cvss/phi_encryption.yaml`
- **Description:** Broken or missing controls prevent assurance that sensitive data is encrypted, violating SOC 2 encryption expectations.
- **Remediation:** Define, test, and monitor encryption-related controls to ensure effective coverage and continuous compliance.

### [HIGH] Multi-Factor
- **Framework:** hipaa
- **Rule:** HIPAA-AUTH-003 — Person or Entity Authentication
- **File:** `app/controllers/api/v1/sessions_controller.rb`
- **Description:** Disabling CSRF protection weakens authentication safeguards and can enable unauthorized actions using an authenticated user’s context in applications accessing ePHI.
- **Remediation:** Re-enable CSRF protection or ensure the controller strictly uses stateless authentication mechanisms that are not vulnerable to CSRF.

### [CRITICAL] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Security
- **File:** `app/controllers/api/v1/sessions_controller.rb`
- **Description:** Disabling CSRF protections weakens authentication mechanisms and exposes authenticated endpoints to forgery attacks.
- **Remediation:** Enable CSRF protections or redesign authentication to use secure, stateless tokens with appropriate safeguards.

### [HIGH] Session Timeout
- **Framework:** hipaa
- **Rule:** HIPAA-SESS-001 — Automatic Logoff
- **File:** `app/controllers/api/v1/sessions_controller.rb`
- **Description:** JWTs without expiration allow indefinite sessions, violating requirements to limit session lifetime and reduce unauthorized access to ePHI.
- **Remediation:** Include expiration (`exp`), issuer, and audience claims in JWTs and enforce token rotation and revocation.

### [CRITICAL] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Security
- **File:** `app/controllers/api/v1/sessions_controller.rb`
- **Description:** Non-expiring authentication tokens increase the risk of unauthorized access and do not meet SOC 2 authentication best practices.
- **Remediation:** Implement short-lived tokens with refresh mechanisms and enforce validation of token claims.

### [CRITICAL] Plaintext Credentials
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-009 — Encryption and Decryption
- **File:** `app/controllers/api/v1/sessions_controller.rb`
- **Description:** Storing JWT access tokens in plaintext constitutes storage of sensitive authentication material without encryption, risking unauthorized access to ePHI.
- **Remediation:** Encrypt tokens at rest or store only hashed representations. Avoid `update_attribute` and enforce validations and secure storage mechanisms.

### [HIGH] Privacy and PII Protection
- **Framework:** soc2
- **Rule:** SOC2-PRIV-001 — Privacy
- **File:** `app/controllers/api/v1/sessions_controller.rb`
- **Description:** Plaintext storage of authentication tokens increases the likelihood of unauthorized disclosure of data that can grant access to sensitive personal and health information.
- **Remediation:** Protect authentication secrets using encryption or hashing and restrict database access to least privilege.

### [HIGH] Audit Logging and Monitoring
- **Framework:** soc2
- **Rule:** SOC2-SEC-005 — Security
- **File:** `app/controllers/api/v1/sessions_controller.rb`
- **Description:** Unstructured logging to stdout bypasses centralized logging, retention, and monitoring controls required for effective auditability under SOC 2.
- **Remediation:** Implement centralized, structured logging with severity levels and monitoring alerts. Ensure logs are reviewed and protected from unauthorized access.

### [CRITICAL] Person or Entity Authentication — MFA
- **Framework:** hipaa
- **Rule:** HIPAA-INT-004 — §164.312(d)
- **File:** `app/controllers/concerns/authenticate_with_otp_two_factor.rb`
- **Description:** The absence of rate limiting or lockout on OTP and backup code attempts weakens multi-factor authentication, allowing brute-force attacks against authentication mechanisms protecting ePHI.
- **Remediation:** Add rate limiting, attempt counters, and temporary lockouts for OTP and backup code failures. Log and alert on excessive failed attempts.

### [CRITICAL] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Security
- **File:** `app/controllers/concerns/authenticate_with_otp_two_factor.rb`
- **Description:** Unlimited OTP attempts violate strong authentication requirements and increase the risk of unauthorized access through brute-force attacks.
- **Remediation:** Enforce throttling, CAPTCHA, and account lockout policies for repeated authentication failures.

### [HIGH] Automatic Logoff — Session Timeout
- **Framework:** hipaa
- **Rule:** HIPAA-SESS-001 — §164.312(a)(2)(iii)
- **File:** `app/controllers/sessions_controller.rb`
- **Description:** JWTs issued without expiration or standard claims do not enforce session termination, increasing the risk of prolonged unauthorized access to ePHI if tokens are compromised.
- **Remediation:** Include exp, iat, and aud claims in JWTs. Enforce short-lived tokens and implement refresh token rotation.

### [CRITICAL] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Security
- **File:** `app/controllers/sessions_controller.rb`
- **Description:** Indefinite JWT validity weakens authentication controls and increases the likelihood of account takeover.
- **Remediation:** Issue time-bound tokens and validate claims on every request.

### [CRITICAL] Encryption and Decryption — Secure Data Storage
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-010 — §164.312(a)(2)(iv)
- **File:** `app/controllers/sessions_controller.rb`
- **Description:** Persisting JWTs in plaintext fails to protect authentication credentials at rest, risking unauthorized access to ePHI if the database is compromised.
- **Remediation:** Store only hashed tokens or encrypt JWTs at rest using strong encryption with proper key management.

### [CRITICAL] Encryption at Rest and in Transit
- **Framework:** soc2
- **Rule:** SOC2-SEC-001 — Security
- **File:** `app/controllers/sessions_controller.rb`
- **Description:** Plaintext storage of bearer tokens violates encryption-at-rest requirements for sensitive authentication data.
- **Remediation:** Apply encryption or hashing to stored tokens and restrict database access.

### [CRITICAL] Access Control — Illegal Access Prevention
- **Framework:** hipaa
- **Rule:** HIPAA-AC-002 — §164.312(a)(1)
- **File:** `config/initializers/cors.rb`
- **Description:** An overly permissive CORS policy allows any origin to interact with APIs, increasing the risk of unauthorized access to systems handling ePHI.
- **Remediation:** Restrict CORS origins, headers, and methods to trusted domains only. Avoid wildcard origins in authenticated contexts.

### [HIGH] Secure Data Transmission
- **Framework:** soc2
- **Rule:** SOC2-SEC-004 — Security
- **File:** `config/initializers/cors.rb`
- **Description:** Permissive CORS weakens controls over cross-origin data transmission and increases the risk of data exfiltration.
- **Remediation:** Define explicit allowed origins and enforce secure cross-origin request policies.

### [CRITICAL] Encryption and Decryption — Secure Data Storage
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-010 — §164.312(a)(2)(iv)
- **File:** `db/migrate/20220917091628_add_jwt_token_to_users.rb`
- **Description:** Storing JWT bearer tokens in plaintext fails to adequately protect authentication credentials that can grant access to ePHI.
- **Remediation:** Hash JWT identifiers or encrypt tokens at rest and enforce expiration and rotation.

### [CRITICAL] Encryption at Rest and in Transit
- **Framework:** soc2
- **Rule:** SOC2-SEC-001 — Security
- **File:** `db/migrate/20220917091628_add_jwt_token_to_users.rb`
- **Description:** Plaintext storage of authentication tokens violates SOC 2 encryption requirements for sensitive data.
- **Remediation:** Encrypt or hash tokens and protect encryption keys using a secrets manager.

### [CRITICAL] Integrity — Plaintext Credentials
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-009 — §164.312(e)(1)
- **File:** `db/migrate/20240319214209_add_access_token_and_repo_type_to_github_uploads.rb`
- **Description:** GitHub access tokens are stored in plaintext, exposing credentials that could be abused to access systems or data.
- **Remediation:** Encrypt access tokens at rest or store them in a secure secrets manager. Rotate existing tokens.

### [CRITICAL] Encryption at Rest and in Transit
- **Framework:** soc2
- **Rule:** SOC2-SEC-001 — Security
- **File:** `db/migrate/20240319214209_add_access_token_and_repo_type_to_github_uploads.rb`
- **Description:** Sensitive access tokens are not protected with encryption at rest, violating SOC 2 security criteria.
- **Remediation:** Apply encryption or external secret storage for access tokens.

### [HIGH] Person or Entity Authentication — Multi-Factor
- **Framework:** hipaa
- **Rule:** HIPAA-AUTH-003 — §164.312(d)
- **File:** `app/views/devise/sessions/two_factor.html.erb`
- **Description:** Plain text OTP entry exposes sensitive authentication data, undermining MFA safeguards required for ePHI access.
- **Remediation:** Use masked password fields for OTP input and prevent caching or autofill.

### [HIGH] Input Validation
- **Framework:** hipaa
- **Rule:** HIPAA-INT-001 — Integrity
- **File:** `app/views/layouts/login.html.erb`
- **Description:** Rendering flash messages with html_safe disables output encoding and allows unvalidated user-controlled input to be rendered as executable HTML/JavaScript, undermining data integrity protections and potentially exposing ePHI through XSS.
- **Remediation:** Remove html_safe usage for flash messages. Rely on Rails' default escaping or explicitly sanitize content using a whitelist-based sanitizer such as sanitize(). Ensure no user-controlled input is rendered without encoding.

### [HIGH] Data Integrity and Validation
- **Framework:** soc2
- **Rule:** SOC2-PI-001 — Processing Integrity
- **File:** `app/views/layouts/login.html.erb`
- **Description:** Failure to properly encode or validate rendered content can allow injection attacks that compromise the integrity of authentication flows and sensitive data processing.
- **Remediation:** Implement consistent output encoding and input validation. Add automated security testing for XSS and enforce secure coding standards for view rendering.

### [HIGH] Data Access Audit
- **Framework:** hipaa
- **Rule:** HIPAA-AUD-001 — Audit Controls
- **File:** `config/initializers/filter_parameter_logging.rb`
- **Description:** Incomplete filtering of request parameters allows ePHI to be written to logs, increasing the risk of unauthorized disclosure and violating minimum necessary and audit control expectations.
- **Remediation:** Expand filter_parameters to include all common ePHI/PII fields (e.g., dob, mrn, diagnosis, medications, email, phone). Periodically review logs and filtering rules for completeness.

### [HIGH] Audit Logging and Monitoring
- **Framework:** soc2
- **Rule:** SOC2-SEC-005 — Security
- **File:** `config/initializers/filter_parameter_logging.rb`
- **Description:** Logging sensitive data without adequate filtering undermines secure logging practices and increases exposure of PII/PHI in monitoring systems.
- **Remediation:** Implement comprehensive sensitive-data redaction in logs and restrict access to log storage. Conduct regular audits of logged fields.

### [CRITICAL] Plaintext Credentials
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-009 — Integrity
- **File:** `app/views/two_factor_settings/edit.html.erb`
- **Description:** Rendering MFA backup codes in plaintext exposes authentication credentials, enabling potential unauthorized access to ePHI if compromised.
- **Remediation:** Display backup codes only once, require re-authentication before viewing, mask codes by default, disable caching, and provide secure download or print handling guidance.

### [CRITICAL] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Security
- **File:** `app/views/two_factor_settings/edit.html.erb`
- **Description:** Exposure of authentication secrets weakens the effectiveness of multi-factor authentication controls required to protect sensitive systems and data.
- **Remediation:** Treat backup codes as secrets: restrict visibility, enforce one-time access, and store and transmit them securely.

### [CRITICAL] Plaintext Credentials
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-009 — Integrity
- **File:** `app/views/two_factor_settings/new.html.erb`
- **Description:** Displaying the OTP shared secret in plaintext exposes a core authentication secret, enabling full bypass of MFA protections if obtained.
- **Remediation:** Avoid rendering OTP secrets in plaintext. Use QR codes only, restrict one-time display, require re-authentication, and prevent caching or logging.

### [CRITICAL] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Security
- **File:** `app/views/two_factor_settings/new.html.erb`
- **Description:** Plaintext exposure of OTP secrets compromises strong authentication mechanisms required to secure access to systems handling sensitive data.
- **Remediation:** Implement secure MFA enrollment patterns and ensure secrets are never exposed beyond initial controlled setup.

### [HIGH] Optimal Cryptography
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-001 — Encryption and Decryption
- **File:** `patterns/released/dotnet/encryption_decryption.yaml`
- **Description:** Non-functional encryption detection rules prevent identification of missing or weak cryptography, increasing the risk that ePHI is not adequately protected.
- **Remediation:** Replace placeholder patterns with functional detection logic and validate rules against representative code samples.

### [HIGH] Audit Logging and Monitoring
- **Framework:** soc2
- **Rule:** SOC2-SEC-005 — Security
- **File:** `patterns/released/dotnet/encryption_decryption.yaml`
- **Description:** Ineffective security detection rules create monitoring blind spots and reduce assurance that encryption controls are in place.
- **Remediation:** Implement effective rule patterns and continuously test the security analysis pipeline.

### [HIGH] Optimal Cryptography
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-001 — Encryption and Decryption
- **File:** `patterns/released/spring_cvss/encryption_decryption.yaml`
- **Description:** Disabled encryption detection rules impair the ability to identify inadequate cryptographic protections for ePHI.
- **Remediation:** Define actionable patterns to detect encryption usage and misconfigurations; validate rule effectiveness regularly.

### [HIGH] Multi-Factor
- **Framework:** hipaa
- **Rule:** HIPAA-AUTH-003 — Person or Entity Authentication
- **File:** `patterns/released/android/user_authentication.yaml`
- **Description:** Disabled authentication detection rules prevent verification that strong authentication mechanisms are implemented to protect ePHI.
- **Remediation:** Implement functional detection patterns for OAuth, Firebase Auth, and MFA, and validate against known implementations.

### [HIGH] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Security
- **File:** `patterns/released/dotnet/user_authentication.yaml`
- **Description:** Non-functional authentication detection rules create blind spots in assessing whether adequate authentication controls exist.
- **Remediation:** Replace placeholder patterns with effective rules and routinely test detection coverage.

### [HIGH] Authorization Enforcement
- **Framework:** hipaa
- **Rule:** HIPAA-AC-003 — Access Control
- **File:** `patterns/released/dotnet_cvss/user_authentication.yaml`
- **Description:** Inoperative authentication and authorization detection rules undermine assurance that access controls required for ePHI are enforced.
- **Remediation:** Implement and validate detection logic for identity, OAuth, and MFA enforcement.

### [HIGH] Authorization for Data Destruction
- **Framework:** hipaa
- **Rule:** HIPAA-AC-006 — Access Control
- **File:** `patterns/released/ios_cvss/authorization_for_destruction.yaml`
- **Description:** Non-functional authorization rules prevent evaluation of controls that restrict destruction or modification of PHI to authorized entities.
- **Remediation:** Replace 'N/A' patterns with enforceable authorization checks and test rule coverage against real code paths.

### [HIGH] Integrity — Input Validation
- **Framework:** hipaa
- **Rule:** HIPAA-INT-001 — §164.312(c)(1)
- **File:** `app/controllers/api/v1/ios/analyzed_results_controller.rb`
- **Description:** User-controlled input is persisted without validation or sanitization, creating risk of stored XSS and unauthorized alteration or exposure of ePHI, violating HIPAA integrity safeguards.
- **Remediation:** Implement strong server-side validations and sanitization (e.g., allowlists, length checks, escaping). Use Rails strong parameters and output encoding in all HTML-rendered contexts.

### [HIGH] Data Integrity and Validation
- **Framework:** soc2
- **Rule:** SOC2-PI-001 — Processing Integrity
- **File:** `app/controllers/api/v1/ios/analyzed_results_controller.rb`
- **Description:** Lack of validation allows malformed or malicious data to be stored, undermining processing integrity and increasing risk of data corruption or exploitation.
- **Remediation:** Enforce input validation, sanitize stored content, and add automated tests to ensure only well-formed data is persisted.

### [HIGH] Integrity — Input Validation
- **Framework:** hipaa
- **Rule:** HIPAA-INT-001 — §164.312(c)(1)
- **File:** `app/controllers/api/v1/ios/user_uploads_controller.rb`
- **Description:** Bypassing model validations allows invalid or malicious data to be stored, undermining integrity controls required to protect ePHI.
- **Remediation:** Remove validate: false, enforce model-level validations, and handle validation errors explicitly in the controller.

### [HIGH] Data Integrity and Validation
- **Framework:** soc2
- **Rule:** SOC2-PI-001 — Processing Integrity
- **File:** `app/controllers/api/v1/ios/user_uploads_controller.rb`
- **Description:** Explicitly disabling validations violates expectations for accurate, complete, and authorized data processing.
- **Remediation:** Require validation checks and add integrity controls ensuring only compliant records are persisted.

### [HIGH] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Security
- **File:** `app/controllers/api/v1/confirmations_controller.rb`
- **Description:** Disabling CSRF protection on a state-changing authentication endpoint weakens safeguards against unauthorized account actions.
- **Remediation:** Re-enable CSRF protection or ensure the endpoint is strictly token-based and inaccessible via browser session cookies.

### [HIGH] Access Control — Illegal Access Prevention
- **Framework:** hipaa
- **Rule:** HIPAA-AC-002 — §164.312(a)(1)
- **File:** `app/controllers/api/v1/confirmations_controller.rb`
- **Description:** Detailed error messages allow attackers to infer account state, increasing risk of unauthorized access attempts.
- **Remediation:** Return generic error messages for authentication failures and log detailed reasons only on the server side.

### [CRITICAL] Integrity — Plaintext Credentials
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-009 — §164.312(a)(2)(iv)
- **File:** `app/controllers/api/v1/github_uploads_controller.rb`
- **Description:** Sensitive access tokens are accepted and likely stored without encryption, risking credential compromise.
- **Remediation:** Encrypt access tokens at rest using strong encryption, store secrets via a managed secrets service, and restrict access.

### [CRITICAL] Encryption at Rest and in Transit
- **Framework:** soc2
- **Rule:** SOC2-SEC-001 — Security
- **File:** `app/controllers/api/v1/github_uploads_controller.rb`
- **Description:** Failure to encrypt sensitive credentials violates encryption requirements for protecting confidential information.
- **Remediation:** Apply field-level encryption and rotate tokens regularly. Avoid logging or exposing secrets.

### [HIGH] Audit Controls — Data Access Audit
- **Framework:** hipaa
- **Rule:** HIPAA-AUD-001 — §164.312(b)
- **File:** `app/controllers/api/v1/licenses_controller.rb`
- **Description:** Logging full responses to stdout risks exposing sensitive operational data in logs beyond minimum necessary use.
- **Remediation:** Remove verbose logging, redact sensitive fields, and use structured logging with access controls.

### [HIGH] Privacy and PII Protection
- **Framework:** soc2
- **Rule:** SOC2-PRIV-001 — Privacy
- **File:** `app/controllers/api/v1/licenses_controller.rb`
- **Description:** Persistent logging of IP addresses and user agents without minimization or retention controls increases PII exposure risk.
- **Remediation:** Justify logging necessity, anonymize or truncate IPs, define retention limits, and document privacy controls.

### [CRITICAL] Access Control — Authorization Enforcement
- **Framework:** hipaa
- **Rule:** HIPAA-AC-003 — §164.312(a)(1)
- **File:** `app/controllers/api/v1/members_controller.rb`
- **Description:** Lack of authorization checks allows unauthorized modification or deletion of user accounts, risking improper access to ePHI.
- **Remediation:** Implement role-based authorization checks (e.g., Pundit/CanCanCan) for update and destroy actions.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Security
- **File:** `app/controllers/api/v1/members_controller.rb`
- **Description:** Insufficient authorization enforcement enables privilege escalation and unauthorized account management.
- **Remediation:** Define and enforce least-privilege access policies and regularly review authorization logic.

### [CRITICAL] OAuth Integrity
- **Framework:** hipaa
- **Rule:** HIPAA-INT-002 — Person or Entity Authentication
- **File:** `app/controllers/api/v1/members_controller.rb`
- **Description:** Allowing mass-assignment of the confirmed_at attribute enables bypass of email verification, undermining the integrity of the authentication process required under HIPAA.
- **Remediation:** Remove confirmed_at from permitted parameters. Ensure account confirmation can only occur through verified email tokens managed by the authentication framework.

### [HIGH] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Security
- **File:** `app/controllers/api/v1/members_controller.rb`
- **Description:** Improper parameter permitting allows users to self-confirm accounts, weakening authentication controls.
- **Remediation:** Enforce strict server-side authentication flows and restrict sensitive authentication attributes from user-controlled input.

### [CRITICAL] Authorization Enforcement
- **Framework:** hipaa
- **Rule:** HIPAA-AC-003 — Access Control
- **File:** `app/controllers/api/v1/members_controller.rb`
- **Description:** Invitation and re-invitation endpoints lack role-based authorization, allowing unauthorized users to provision access.
- **Remediation:** Enforce admin-only authorization checks (e.g., before_action with role validation) on invitation-related actions.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Security
- **File:** `app/controllers/api/v1/members_controller.rb`
- **Description:** Missing authorization allows any authenticated user to manage invitations, violating least-privilege principles.
- **Remediation:** Implement RBAC checks ensuring only authorized roles can invite or re-invite users.

### [HIGH] Unique User Identification
- **Framework:** hipaa
- **Rule:** HIPAA-UID-001 — Access Control
- **File:** `app/controllers/api/v1/members_controller.rb`
- **Description:** API responses reveal whether a user exists based on email, enabling user enumeration and privacy leakage.
- **Remediation:** Standardize responses for invite requests regardless of user existence and avoid disclosing account status.

### [HIGH] Privacy and PII Protection
- **Framework:** soc2
- **Rule:** SOC2-PRIV-001 — Privacy
- **File:** `app/controllers/api/v1/members_controller.rb`
- **Description:** User enumeration via email disclosure exposes PII and violates privacy protections.
- **Remediation:** Return generic success messages and log detailed outcomes internally only.

### [CRITICAL] Illegal Access Prevention
- **Framework:** hipaa
- **Rule:** HIPAA-AC-002 — Access Control
- **File:** `app/controllers/api/v1/organizations_controller.rb`
- **Description:** Organization data is accessible without authentication or authorization, enabling unauthorized access.
- **Remediation:** Require authentication (e.g., authenticate_user!) and enforce tenant-based authorization for all actions.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Security
- **File:** `app/controllers/api/v1/organizations_controller.rb`
- **Description:** Unauthenticated access to organization endpoints violates access control requirements.
- **Remediation:** Implement authentication and authorization middleware for all organization resources.

### [HIGH] Access Control Exception
- **Framework:** hipaa
- **Rule:** HIPAA-AC-004 — Access Control
- **File:** `app/controllers/api/v1/organizations_controller.rb`
- **Description:** CSRF protection is disabled without compensating controls, increasing risk of unauthorized state-changing requests.
- **Remediation:** Re-enable CSRF protection or ensure token-based authentication with strict origin and rate controls.

### [HIGH] Unique User Identification
- **Framework:** hipaa
- **Rule:** HIPAA-UID-001 — Access Control
- **File:** `app/controllers/api/v1/passwords_controller.rb`
- **Description:** Password reset responses differ based on email existence, enabling user enumeration.
- **Remediation:** Return uniform responses for password reset requests regardless of account existence.

### [HIGH] Privacy and PII Protection
- **Framework:** soc2
- **Rule:** SOC2-PRIV-001 — Privacy
- **File:** `app/controllers/api/v1/passwords_controller.rb`
- **Description:** Email enumeration exposes user privacy and account existence.
- **Remediation:** Mask account existence through consistent API messaging and monitoring.

### [HIGH] Access Control Exception
- **Framework:** hipaa
- **Rule:** HIPAA-AC-004 — Access Control
- **File:** `app/controllers/api/v1/passwords_controller.rb`
- **Description:** CSRF protection is disabled for password reset actions, increasing risk of request forgery or abuse.
- **Remediation:** Re-enable CSRF protection or enforce strong token validation and rate limiting.

### [CRITICAL] Plaintext Credentials
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-009 — Integrity
- **File:** `app/controllers/api/v1/payment_api_controller.rb`
- **Description:** Stripe payment tokens are logged in plaintext, exposing sensitive financial credentials.
- **Remediation:** Remove sensitive data from logs and implement log redaction for payment-related fields.

### [HIGH] Audit Logging and Monitoring
- **Framework:** soc2
- **Rule:** SOC2-SEC-005 — Security
- **File:** `app/controllers/api/v1/payment_api_controller.rb`
- **Description:** Logging sensitive credentials violates secure logging practices.
- **Remediation:** Sanitize logs and ensure secrets and tokens are never logged.

### [HIGH] Data Access Audit
- **Framework:** hipaa
- **Rule:** HIPAA-AUD-001 — Audit Controls
- **File:** `app/controllers/api/v1/payment_api_controller.rb`
- **Description:** Verbose exception logging may capture sensitive identifiers and system data.
- **Remediation:** Limit exception logging in production and redact sensitive information.

### [CRITICAL] Access Control — Authorization Enforcement
- **Framework:** hipaa
- **Rule:** HIPAA-AC-003 — §164.312(a)(1)
- **File:** `app/controllers/api/v1/payment_methods_controller.rb`
- **Description:** Disabling CSRF protection while using cookie-based authentication weakens authorization enforcement and allows authenticated actions to be performed without user intent.
- **Remediation:** Re-enable CSRF protection for cookie-authenticated controllers. Use per-request CSRF tokens or switch to stateless token-based authentication for APIs.

### [HIGH] Integrity — Input Validation
- **Framework:** hipaa
- **Rule:** HIPAA-INT-001 — §164.312(c)(1)
- **File:** `app/controllers/api/v1/payment_methods_controller.rb`
- **Description:** User-controlled parameters are sent directly to an external payment processor without validation, risking integrity and availability of downstream systems.
- **Remediation:** Use strong parameter filtering and explicit validation for card tokens. Reject malformed or unexpected input before invoking third-party APIs.

### [HIGH] Privacy and PII Protection
- **Framework:** soc2
- **Rule:** SOC2-PRIV-001 — Privacy
- **File:** `app/controllers/api/v1/payment_methods_controller.rb`
- **Description:** Returning the full Stripe card object exposes more financial metadata than necessary, increasing the risk of sensitive data leakage.
- **Remediation:** Limit API responses to only required fields (e.g., brand and last4). Use serializers or explicit JSON shaping to enforce data minimization.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Security — Access Controls
- **File:** `app/controllers/api/v1/promotional_codes_controller.rb`
- **Description:** Public access to the promotional codes endpoint lacks authentication and authorization, violating access control expectations.
- **Remediation:** Require authentication for the endpoint and enforce role-based authorization to restrict access to permitted users only.

### [CRITICAL] Access Control — Illegal Access Prevention
- **Framework:** hipaa
- **Rule:** HIPAA-AC-002 — §164.312(a)(1)
- **File:** `app/controllers/api/v1/registrations_controller.rb`
- **Description:** Fully disabling CSRF protections can allow unauthorized or unintended actions, undermining safeguards against illegal access.
- **Remediation:** Re-enable CSRF protections or ensure the endpoint is strictly stateless and inaccessible from browser contexts with proper CORS controls.

### [HIGH] Person or Entity Authentication — OAuth
- **Framework:** hipaa
- **Rule:** HIPAA-AUTH-001 — §164.312(d)
- **File:** `app/controllers/api/v1/registrations_controller.rb`
- **Description:** Allowing expired invitation tokens bypasses authentication integrity and increases the risk of unauthorized account creation.
- **Remediation:** Enforce invitation token expiration checks and invalidate tokens after use. Log and monitor failed or expired token usage.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Security — Access Controls
- **File:** `app/controllers/api/v1/roles_controller.rb`
- **Description:** Unauthenticated access to role listings exposes internal authorization structure and violates access control principles.
- **Remediation:** Require authentication and restrict role enumeration to authorized administrative users only.

### [HIGH] Integrity — Input Validation
- **Framework:** hipaa
- **Rule:** HIPAA-INT-001 — §164.312(c)(1)
- **File:** `app/controllers/api/v1/rules_controller.rb`
- **Description:** Unsafe YAML deserialization allows unvalidated external input to instantiate arbitrary objects, undermining application integrity safeguards required under HIPAA.
- **Remediation:** Replace YAML.load_file with YAML.safe_load and explicitly permit required classes. Ensure YAML files are immutable at runtime and protected in the deployment pipeline.

### [HIGH] Data Integrity and Validation
- **Framework:** soc2
- **Rule:** SOC2-PI-001 — Processing Integrity
- **File:** `app/controllers/api/v1/rules_controller.rb`
- **Description:** Unsafe deserialization bypasses integrity controls and may result in unauthorized code execution, violating SOC 2 integrity expectations.
- **Remediation:** Implement safe deserialization practices and restrict accepted data structures. Add integrity checks and code review controls for configuration files.

### [HIGH] Audit Controls — Data Access Audit
- **Framework:** hipaa
- **Rule:** HIPAA-AUD-001 — §164.312(b)
- **File:** `app/controllers/api/v1/subscriptions_controller.rb`
- **Description:** Logging full exception backtraces and exposing raw error messages risks disclosure of sensitive system or ePHI-related context in logs and API responses.
- **Remediation:** Sanitize logs to remove sensitive data, avoid printing stack traces to stdout, and return generic error messages to clients. Use structured, access-controlled logging.

### [HIGH] Audit Logging and Monitoring
- **Framework:** soc2
- **Rule:** SOC2-SEC-005 — Security Monitoring
- **File:** `app/controllers/api/v1/subscriptions_controller.rb`
- **Description:** Excessive logging of exceptions without redaction can expose sensitive information and indicates inadequate log hygiene.
- **Remediation:** Implement centralized logging with redaction, severity-based logging levels, and restrict access to logs.

### [CRITICAL] Encryption and Decryption — Secure Data Storage
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-010 — §164.312(a)(2)(iv)
- **File:** `app/controllers/api/v1/subscriptions_controller.rb`
- **Description:** Sensitive Google Play purchase tokens are stored in plaintext, violating HIPAA requirements to protect sensitive credentials at rest.
- **Remediation:** Encrypt purchase tokens using strong, industry-standard encryption. Manage encryption keys securely using a KMS or secrets manager.

### [CRITICAL] Encryption at Rest and in Transit
- **Framework:** soc2
- **Rule:** SOC2-SEC-001 — Confidentiality
- **File:** `app/controllers/api/v1/subscriptions_controller.rb`
- **Description:** Storing sensitive credentials without encryption violates SOC 2 confidentiality and security principles.
- **Remediation:** Apply encryption at rest for sensitive fields and ensure keys are rotated and access-controlled.

### [CRITICAL] Access Control — Illegal Access Prevention
- **Framework:** hipaa
- **Rule:** HIPAA-AC-002 — §164.312(a)(1)
- **File:** `app/controllers/api/v1/suggestions_controller.rb`
- **Description:** Disabling CSRF protection weakens safeguards against unauthorized actions performed on behalf of authenticated users.
- **Remediation:** Re-enable CSRF protection or enforce token-based authentication for API endpoints not using browser sessions.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Access Controls
- **File:** `app/controllers/api/v1/suggestions_controller.rb`
- **Description:** Lack of CSRF protection undermines access control mechanisms intended to prevent unauthorized requests.
- **Remediation:** Implement CSRF defenses or require stateless authentication with proper authorization checks.

### [HIGH] Audit Controls — User Event Tracking
- **Framework:** hipaa
- **Rule:** HIPAA-AUD-002 — §164.312(b)
- **File:** `app/controllers/api/v1/suggestions_controller.rb`
- **Description:** Returning raw exception messages to clients can disclose internal system details that should be protected and audited.
- **Remediation:** Return generic error responses and log detailed errors securely with access controls.

### [HIGH] Integrity — Input Validation
- **Framework:** hipaa
- **Rule:** HIPAA-INT-001 — §164.312(c)(1)
- **File:** `app/controllers/api/v1/suggestions_controller.rb`
- **Description:** Insufficient sanitization of HTML content allows injection of malicious scripts, threatening data and application integrity.
- **Remediation:** Use a robust HTML sanitization library to strip or escape dangerous tags and attributes before returning content.

### [HIGH] Data Integrity and Validation
- **Framework:** soc2
- **Rule:** SOC2-PI-001 — Processing Integrity
- **File:** `app/controllers/api/v1/suggestions_controller.rb`
- **Description:** Failure to properly sanitize user-controlled HTML content violates SOC 2 integrity and validation expectations.
- **Remediation:** Implement server-side sanitization and output encoding for all dynamic content.

### [CRITICAL] Access Control — Authorization Enforcement
- **Framework:** hipaa
- **Rule:** HIPAA-AC-003 — §164.312(a)(1)
- **File:** `app/controllers/api/v1/user_controller.rb`
- **Description:** Implicitly exposing the full user object without attribute-level restrictions risks unauthorized disclosure of ePHI.
- **Remediation:** Use explicit serializers or presenters to limit returned fields to the minimum necessary.

### [HIGH] Privacy and PII Protection
- **Framework:** soc2
- **Rule:** SOC2-PRIV-001 — Privacy
- **File:** `app/controllers/api/v1/user_controller.rb`
- **Description:** Excessive exposure of user attributes may disclose PII beyond intended purposes.
- **Remediation:** Implement data minimization through explicit API response schemas.

### [CRITICAL] Person or Entity Authentication — Multi-Factor
- **Framework:** hipaa
- **Rule:** HIPAA-AUTH-003 — §164.312(d)
- **File:** `app/controllers/api/v1/user_controller.rb`
- **Description:** Allowing destructive actions without step-up or multi-factor authentication increases risk of unauthorized account deletion.
- **Remediation:** Require re-authentication, password confirmation, or MFA before permitting account deletion.

### [CRITICAL] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Authentication
- **File:** `app/controllers/api/v1/user_controller.rb`
- **Description:** Lack of strong authentication for sensitive operations violates SOC 2 authentication requirements.
- **Remediation:** Implement step-up authentication and MFA for high-risk actions.

### [HIGH] Audit Controls — Data Access Audit
- **Framework:** hipaa
- **Rule:** HIPAA-AUD-001 — §164.312(b)
- **File:** `app/controllers/api/v1/user_uploads_controller.rb`
- **Description:** Logging full exception messages and stack traces may expose ePHI or sensitive system details in logs.
- **Remediation:** Reduce log verbosity, redact sensitive data, and route logs to secure, access-controlled logging systems.

### [HIGH] Audit Logging and Monitoring
- **Framework:** soc2
- **Rule:** SOC2-SEC-005 — Security Monitoring
- **File:** `app/controllers/api/v1/user_uploads_controller.rb`
- **Description:** Insecure logging practices increase the risk of sensitive data exposure and weaken monitoring controls.
- **Remediation:** Adopt secure logging standards with redaction and controlled access.

### [CRITICAL] Access Control — Illegal Access Prevention
- **Framework:** hipaa
- **Rule:** HIPAA-AC-002 — §164.312(a)(1)
- **File:** `app/controllers/api/v1/user_uploads_controller.rb`
- **Description:** Reading arbitrary file paths without validation enables unauthorized access to sensitive files and ePHI.
- **Remediation:** Validate and constrain file paths to a dedicated directory, and enforce authorization checks before file access.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Access Controls
- **File:** `app/controllers/api/v1/user_uploads_controller.rb`
- **Description:** Arbitrary file read capabilities indicate insufficient authorization and resource access controls.
- **Remediation:** Restrict file access to whitelisted directories and implement strict authorization and validation logic.

### [HIGH] Integrity — Input Validation
- **Framework:** hipaa
- **Rule:** HIPAA-INT-001 — §164.312(c)(1)
- **File:** `app/controllers/api/v2/user_uploads_controller.rb`
- **Description:** Using YAML.load_file allows unsafe deserialization and arbitrary object instantiation, which can compromise system integrity and potentially expose or alter ePHI if attacker-controlled YAML is loaded.
- **Remediation:** Replace YAML.load_file with YAML.safe_load and explicitly whitelist permitted classes. Validate file paths and ensure rule files are not user-influenced.

### [HIGH] Data Integrity and Validation
- **Framework:** soc2
- **Rule:** SOC2-PI-001 — Processing Integrity
- **File:** `app/controllers/api/v2/user_uploads_controller.rb`
- **Description:** Unsafe deserialization undermines application data integrity controls and can result in unauthorized code execution affecting system reliability.
- **Remediation:** Enforce safe deserialization methods, validate inputs, and restrict file loading to trusted, immutable sources.

### [CRITICAL] Access Control — Illegal Access Prevention
- **Framework:** hipaa
- **Rule:** HIPAA-AC-002 — §164.312(a)(1)
- **File:** `app/controllers/api/v2/user_uploads_controller.rb`
- **Description:** Unvalidated path construction enables directory traversal, potentially allowing unauthorized access to files containing credentials or ePHI.
- **Remediation:** Validate and normalize pattern_path against an allowlist, reject traversal characters, and enforce directory boundaries using realpath checks.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Security
- **File:** `app/controllers/api/v2/user_uploads_controller.rb`
- **Description:** Path traversal weaknesses allow bypass of intended authorization boundaries for filesystem access.
- **Remediation:** Implement strict input validation and ensure file access is constrained to authorized directories.

### [CRITICAL] Access Control — Authorization Enforcement
- **Framework:** hipaa
- **Rule:** HIPAA-AC-003 — §164.312(a)(1)
- **File:** `app/controllers/api/v2/user_uploads_controller.rb`
- **Description:** Arbitrary file reads can expose system files or ePHI without proper authorization enforcement.
- **Remediation:** Restrict readable paths to an allowlisted base directory and validate ownership and authorization before reading files.

### [HIGH] Encryption at Rest and in Transit
- **Framework:** soc2
- **Rule:** SOC2-SEC-001 — Security
- **File:** `app/controllers/api/v2/user_uploads_controller.rb`
- **Description:** Uncontrolled file access increases the likelihood of unauthorized disclosure of sensitive data stored at rest.
- **Remediation:** Apply access controls to file reads and ensure sensitive files are encrypted and inaccessible to application users.

### [CRITICAL] Access Control — Authorization Enforcement
- **Framework:** hipaa
- **Rule:** HIPAA-AC-003 — §164.312(a)(1)
- **File:** `app/controllers/api/v2/user_uploads_controller.rb`
- **Description:** Failure to scope UserUpload records to the current user allows unauthorized access to other users’ reports containing PHI.
- **Remediation:** Scope queries by current_user (e.g., current_user.user_uploads.find(params[:id])) and add authorization checks.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Security
- **File:** `app/controllers/api/v2/user_uploads_controller.rb`
- **Description:** Broken object-level authorization allows authenticated users to access resources they do not own.
- **Remediation:** Implement consistent object-level authorization and automated tests for access control enforcement.

### [CRITICAL] Integrity — Plaintext Credentials
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-009 — §164.312(a)(2)(iv)
- **File:** `app/controllers/webhooks/google/event_notifications_google_controller.rb`
- **Description:** Storing long-lived service account private keys on disk risks credential compromise and unauthorized access to systems interacting with ePHI.
- **Remediation:** Remove key files from the codebase and load credentials securely via environment variables or a managed secrets service.

### [CRITICAL] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Security
- **File:** `app/controllers/webhooks/google/event_notifications_google_controller.rb`
- **Description:** Hardcoded credentials undermine secure authentication and secret management practices.
- **Remediation:** Adopt centralized secrets management and rotate compromised keys immediately.

### [HIGH] Audit Controls — Data Access Audit
- **Framework:** hipaa
- **Rule:** HIPAA-AUD-001 — §164.312(b)
- **File:** `app/controllers/webhooks/google/event_notifications_google_controller.rb`
- **Description:** Logging full webhook payloads and decoded messages may expose sensitive data in logs beyond the minimum necessary.
- **Remediation:** Redact or omit sensitive fields from logs and limit logging to non-sensitive metadata.

### [HIGH] Privacy and PII Protection
- **Framework:** soc2
- **Rule:** SOC2-PRIV-001 — Privacy
- **File:** `app/controllers/webhooks/google/event_notifications_google_controller.rb`
- **Description:** Excessive logging increases the risk of sensitive data exposure through log access.
- **Remediation:** Implement log sanitization and enforce privacy-aware logging standards.

### [HIGH] Audit Controls — User Event Tracking
- **Framework:** hipaa
- **Rule:** HIPAA-AUD-002 — §164.312(b)
- **File:** `app/controllers/webhooks/google/event_notifications_google_controller.rb`
- **Description:** Logging decoded JWTs exposes sensitive authentication claims that should not be persisted.
- **Remediation:** Avoid logging token contents; log only verification success or failure.

### [HIGH] Privacy and PII Protection
- **Framework:** soc2
- **Rule:** SOC2-PRIV-001 — Privacy
- **File:** `app/controllers/webhooks/google/event_notifications_google_controller.rb`
- **Description:** Logging full API responses containing customer-linked subscription data increases exposure of sensitive information.
- **Remediation:** Restrict logs to non-sensitive status indicators and correlation IDs.

### [CRITICAL] Authorization Enforcement
- **Framework:** hipaa
- **Rule:** HIPAA-AC-003 — Access Control — Authorization Enforcement
- **File:** `app/controllers/analyzed_results_controller.rb`
- **Description:** UserUpload records containing ePHI are accessed directly by ID without verifying that the authenticated user is authorized to access the resource, enabling IDOR and unauthorized access to ePHI.
- **Remediation:** Scope UserUpload queries to current_user (e.g., current_user.user_uploads.find). Enforce authorization checks using a policy framework such as Pundit or CanCanCan.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Authorization and Access Controls
- **File:** `app/controllers/analyzed_results_controller.rb`
- **Description:** Lack of ownership validation allows unauthorized users to access resources belonging to others, violating SOC 2 access control requirements.
- **Remediation:** Implement resource-level authorization checks and enforce least-privilege access for all controller actions.

### [CRITICAL] Authorization for Data Destruction
- **Framework:** hipaa
- **Rule:** HIPAA-AC-006 — Access Control — Authorization for Data Destruction
- **File:** `app/controllers/analyzed_results_controller.rb`
- **Description:** AnalyzedResult records containing ePHI can be deleted without verifying user authorization, allowing unauthorized destruction of ePHI.
- **Remediation:** Ensure destroy actions verify ownership and authorization before deletion. Scope queries to authorized parent resources and enforce policy checks.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Authorization and Access Controls
- **File:** `app/controllers/analyzed_results_controller.rb`
- **Description:** Improper authorization allows deletion of other users’ data, violating SOC 2 access control expectations.
- **Remediation:** Restrict destructive actions to authorized users and implement consistent authorization checks.

### [HIGH] Audit Controls
- **Framework:** hipaa
- **Rule:** HIPAA-AUD-001 — Audit Controls — Data Access Audit
- **File:** `app/controllers/api_controller.rb`
- **Description:** Logging raw exception messages and backtraces may expose ePHI in logs, violating HIPAA audit control and minimum necessary requirements.
- **Remediation:** Sanitize logs to remove sensitive data, log structured error identifiers instead of raw messages, and restrict log access.

### [HIGH] Audit Logging and Monitoring
- **Framework:** soc2
- **Rule:** SOC2-SEC-005 — Audit Logging and Monitoring
- **File:** `app/controllers/api_controller.rb`
- **Description:** Excessive logging of sensitive exception data increases the risk of unauthorized disclosure through logs.
- **Remediation:** Implement log filtering and centralized logging with redaction of sensitive fields.

### [CRITICAL] Illegal Access Prevention
- **Framework:** hipaa
- **Rule:** HIPAA-AC-002 — Access Control — Illegal Access Prevention
- **File:** `app/controllers/application_controller.rb`
- **Description:** Disabling CSRF protection for JSON requests can allow attackers to perform unauthorized actions on behalf of authenticated users, risking illegal access to ePHI.
- **Remediation:** Re-enable CSRF protection for authenticated JSON endpoints or use stateless token-based authentication for APIs.

### [CRITICAL] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Authentication Mechanisms
- **File:** `app/controllers/application_controller.rb`
- **Description:** Weak CSRF protections undermine the integrity of authentication mechanisms.
- **Remediation:** Ensure proper CSRF defenses or migrate to secure API authentication patterns.

### [CRITICAL] Authorization Exception Handling
- **Framework:** hipaa
- **Rule:** HIPAA-AC-001 — Access Control — Authorization Exception Handling
- **File:** `app/controllers/application_controller.rb`
- **Description:** Authorization failures do not halt execution, potentially allowing unauthorized access to admin-only functionality handling ePHI.
- **Remediation:** Explicitly halt execution after redirects (e.g., return or head) and enforce authorization consistently.

### [CRITICAL] Authorization Enforcement
- **Framework:** hipaa
- **Rule:** HIPAA-AC-003 — Access Control — Authorization Enforcement
- **File:** `app/controllers/application_controller.rb`
- **Description:** Permitting users to set organization identifiers allows privilege escalation and unauthorized access to other organizations’ ePHI.
- **Remediation:** Remove sensitive authorization attributes from permitted parameters and manage organizational assignment server-side.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Authorization and Access Controls
- **File:** `app/controllers/application_controller.rb`
- **Description:** Mass assignment of authorization attributes violates least-privilege access controls.
- **Remediation:** Restrict parameter whitelisting and enforce role and organization assignment through trusted workflows.

### [CRITICAL] Plaintext Credentials
- **Framework:** hipaa
- **Rule:** HIPAA-ENC-009 — Integrity — Plaintext Credentials
- **File:** `app/controllers/github_uploads_controller.rb`
- **Description:** OAuth access tokens are accepted and potentially stored without encryption, exposing sensitive credentials.
- **Remediation:** Encrypt tokens at rest using strong encryption and store secrets in a secure secrets manager.

### [CRITICAL] Encryption at Rest and in Transit
- **Framework:** soc2
- **Rule:** SOC2-SEC-001 — Encryption at Rest and in Transit
- **File:** `app/controllers/github_uploads_controller.rb`
- **Description:** Lack of encryption for stored access tokens violates SOC 2 encryption requirements.
- **Remediation:** Apply encryption for sensitive credentials and limit access to authorized services.

### [HIGH] Input Validation
- **Framework:** hipaa
- **Rule:** HIPAA-INT-001 — Integrity — Input Validation
- **File:** `app/controllers/home_controller.rb`
- **Description:** User-controlled file paths are not validated, allowing path traversal and unauthorized access to files containing ePHI.
- **Remediation:** Validate and sanitize file paths, restrict access to allowed directories, and enforce authorization.

### [HIGH] Data Integrity and Validation
- **Framework:** soc2
- **Rule:** SOC2-PI-001 — Data Integrity and Validation
- **File:** `app/controllers/home_controller.rb`
- **Description:** Lack of input validation enables arbitrary file reads, compromising data integrity and confidentiality.
- **Remediation:** Implement strict input validation and access controls for file operations.

### [CRITICAL] Authorization Enforcement
- **Framework:** hipaa
- **Rule:** HIPAA-AC-003 — Access Control — Authorization Enforcement
- **File:** `app/controllers/home_controller.rb`
- **Description:** UserUpload resources are accessed by raw ID without ownership checks, enabling unauthorized actions on ePHI.
- **Remediation:** Scope queries to current_user-owned resources and enforce authorization policies.

### [HIGH] Integrity — Input Validation
- **Framework:** hipaa
- **Rule:** HIPAA-INT-001 — §164.312(c)(1)
- **File:** `app/controllers/home_controller.rb`
- **Description:** Using YAML.load_file allows unsafe object deserialization if file contents or paths are attacker-controlled, undermining integrity protections required for systems handling regulated data.
- **Remediation:** Replace YAML.load_file with YAML.safe_load and explicitly whitelist permitted classes. Validate and constrain file paths before loading.

### [HIGH] Data Integrity and Validation
- **Framework:** soc2
- **Rule:** SOC2-PI-001 — Processing Integrity
- **File:** `app/controllers/home_controller.rb`
- **Description:** Unsafe deserialization can allow unauthorized modification or execution of application logic, violating SOC 2 requirements for processing integrity.
- **Remediation:** Implement safe deserialization, enforce strict schema validation, and restrict file system access to trusted directories.

### [CRITICAL] Access Control — Illegal Access Prevention
- **Framework:** hipaa
- **Rule:** HIPAA-AC-002 — §164.312(a)(1)
- **File:** `app/controllers/home_controller.rb`
- **Description:** Unvalidated user-controlled paths may allow directory traversal, enabling access to unauthorized files and bypassing access controls.
- **Remediation:** Normalize and validate pattern_path against an allowlist. Prevent relative paths and enforce directory boundaries.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Security
- **File:** `app/controllers/home_controller.rb`
- **Description:** Directory traversal risks weaken logical access controls required under SOC 2.
- **Remediation:** Constrain file access to authorized locations and implement centralized authorization checks.

### [CRITICAL] Person or Entity Authentication — Multi-Factor
- **Framework:** hipaa
- **Rule:** HIPAA-AUTH-003 — §164.312(d)
- **File:** `app/controllers/registrations_controller.rb`
- **Description:** Disabling CSRF protection on registration endpoints without compensating controls allows unauthorized account creation, violating authentication safeguards.
- **Remediation:** Re-enable CSRF protection or require strong alternative authentication such as OAuth tokens or signed requests.

### [CRITICAL] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Security
- **File:** `app/controllers/registrations_controller.rb`
- **Description:** Lack of request authentication exposes account creation to unauthorized actions.
- **Remediation:** Enforce CSRF tokens, CAPTCHA, or authenticated API mechanisms for registration.

### [HIGH] Audit Controls — Data Access Audit
- **Framework:** hipaa
- **Rule:** HIPAA-AUD-001 — §164.312(b)
- **File:** `app/controllers/subscriptions_controller.rb`
- **Description:** Printing full exception backtraces can expose sensitive context in logs, violating audit and minimum necessary standards.
- **Remediation:** Log sanitized error messages and store detailed traces only in secured, access-controlled logging systems.

### [HIGH] Audit Logging and Monitoring
- **Framework:** soc2
- **Rule:** SOC2-SEC-005 — Security
- **File:** `app/controllers/subscriptions_controller.rb`
- **Description:** Uncontrolled logging of backtraces may expose sensitive data and weaken monitoring controls.
- **Remediation:** Implement structured logging with redaction and restrict log access.

### [HIGH] Access Control — Access Control Exception
- **Framework:** hipaa
- **Rule:** HIPAA-AC-004 — §164.312(a)
- **File:** `app/controllers/subscriptions_controller.rb`
- **Description:** Exposing raw exception messages to users may reveal internal system details, undermining access control safeguards.
- **Remediation:** Display generic user-friendly error messages and log detailed errors internally.

### [HIGH] Audit Logging and Monitoring
- **Framework:** soc2
- **Rule:** SOC2-SEC-005 — Security
- **File:** `app/controllers/subscriptions_controller.rb`
- **Description:** Direct database updates bypass monitoring and audit trails required by SOC 2.
- **Remediation:** Ensure all privilege and entitlement changes are logged and reviewable.

### [HIGH] Audit Controls — Data Access Audit
- **Framework:** hipaa
- **Rule:** HIPAA-AUD-001 — §164.312(b)
- **File:** `app/controllers/suggestions_controller.rb`
- **Description:** Logging full request parameters may capture ePHI, violating minimum necessary and audit control requirements.
- **Remediation:** Remove params.inspect logging or implement parameter filtering and redaction.

### [HIGH] Privacy and PII Protection
- **Framework:** soc2
- **Rule:** SOC2-PRIV-001 — Privacy
- **File:** `app/controllers/suggestions_controller.rb`
- **Description:** Logging sensitive parameters risks unauthorized disclosure of PII or PHI.
- **Remediation:** Adopt privacy-aware logging policies and mask sensitive fields.

### [CRITICAL] Person or Entity Authentication — Multi-Factor
- **Framework:** hipaa
- **Rule:** HIPAA-AUTH-003 — §164.312(d)
- **File:** `app/controllers/two_factor_settings_controller.rb`
- **Description:** Allowing 2FA to be disabled without re-authentication weakens required authentication controls.
- **Remediation:** Require password re-entry or OTP verification before disabling two-factor authentication.

### [CRITICAL] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Security
- **File:** `app/controllers/two_factor_settings_controller.rb`
- **Description:** Weak controls around 2FA management violate strong authentication expectations.
- **Remediation:** Enforce step-up authentication for sensitive security setting changes.

### [CRITICAL] Person or Entity Authentication — Multi-Factor
- **Framework:** hipaa
- **Rule:** HIPAA-AUTH-003 — §164.312(d)
- **File:** `app/controllers/two_factor_settings_controller.rb`
- **Description:** Lack of rate limiting on OTP verification enables brute-force attacks against authentication factors.
- **Remediation:** Implement rate limiting, attempt counters, and temporary lockouts for OTP verification.

### [CRITICAL] Authentication Mechanisms
- **Framework:** soc2
- **Rule:** SOC2-SEC-002 — Security
- **File:** `app/controllers/two_factor_settings_controller.rb`
- **Description:** Brute-forceable OTP flows weaken authentication reliability under SOC 2.
- **Remediation:** Add throttling, monitoring, and alerting for failed OTP attempts.

### [CRITICAL] Access Control — Authorization Enforcement
- **Framework:** hipaa
- **Rule:** HIPAA-AC-003 — §164.312(a)(1)
- **File:** `app/controllers/user_uploads_controller.rb`
- **Description:** Accessing records without scoping to the current user allows unauthorized access to other users’ uploads, including potential ePHI.
- **Remediation:** Scope queries to current_user (e.g., current_user.user_uploads.find) and enforce authorization checks.

### [HIGH] Authorization and Access Controls
- **Framework:** soc2
- **Rule:** SOC2-SEC-003 — Security
- **File:** `app/controllers/user_uploads_controller.rb`
- **Description:** Insecure direct object references violate SOC 2 access control principles.
- **Remediation:** Implement object-level authorization and consistent access control enforcement.

### [HIGH] Integrity — Input Validation
- **Framework:** hipaa
- **Rule:** HIPAA-INT-001 — §164.312(c)(1)
- **File:** `app/controllers/user_uploads_controller.rb`
- **Description:** The controller executes a shell command using string interpolation without strict validation or sanitization of inputs. This creates a command injection risk, which violates HIPAA's integrity control requirements to protect systems from improper alteration or destruction due to invalid or malicious input.
- **Remediation:** Avoid invoking shell commands with interpolated variables. Replace backticks with safe Ruby filesystem APIs (e.g., FileUtils.rm_rf with validated paths). Enforce strict type checking and allowlists on any user-influenced values before performing destructive operations.

### [HIGH] Data Integrity and Validation
- **Framework:** soc2
- **Rule:** SOC2-PI-001 — Processing Integrity — Input Validation
- **File:** `app/controllers/user_uploads_controller.rb`
- **Description:** Executing operating system commands with unsanitized or weakly validated inputs undermines processing integrity and exposes the system to unauthorized manipulation or destruction of data, contrary to SOC 2 requirements for accurate and valid processing.
- **Remediation:** Implement strong input validation and eliminate shell execution where possible. Use built-in language libraries for file and directory management, apply least-privilege execution contexts, and add automated tests to ensure invalid inputs cannot trigger destructive behavior.

## 📊 Token Usage

- **Total:** 229,917 / 2,000,000
- **Utilization:** 11.5%
- **Files Analyzed:** 149
