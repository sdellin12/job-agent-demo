# Secure Development and Vulnerability Policy

**Status:** Mandatory engineering policy for all future code, configuration, infrastructure, workflows, example scripts and AI-generated contributions in this repository. **Baseline updated:** 2026-10-24. This is a requirements document, **not** proof that the software has passed an audit or that controls have been implemented. A control is "verified" only with recorded test evidence.

## 1. Standards and scope

Use the [OWASP Top 10:2025](https://top10.owasp.org/2025/) to identify major web risks, [OWASP ASVS 5.0.0](https://asvs.owasp.org/) as a detailed, testable verification baseline for applicable web/API features, and the [OWASP API Security Top 10:2023](https://owasp.org/API-Security/editions/2023/en/0x11-t10/) for APIs. Use [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/) for implementation patterns. Where applicable also use [OWASP MASVS](https://mas.owasp.org/MASVS/), [OWASP LLM Top 10:2025](https://genai.owasp.org/llm-top-10/), and [OWASP SAMM](https://owaspsamm.org/).

- Identify applicable controls by feature, assets and deployment model; **do not** treat the Top 10 awareness list as a complete security checklist. Document ASVS 5.0 requirement IDs for web/API controls when implementation begins; aim at ASVS Level 1 as a minimum, with stronger controls based on threat modeling.
- Apply secure-by-default, least privilege, deny-by-default, defense in depth, fail-closed authorization, minimization of personal data, safe failure and reproducible builds. Security and privacy are part of feature acceptance, not post-release polish.
- Track each applicable control as `implemented + tested`, `planned`, `not applicable (reason)`, or `exception (owner, reason, expiry, mitigation)`. **Never** claim OWASP compliance merely because this file exists.
- Treat any input from a browser, API, filesystem, message queue, plug-in, third-party response, fetched webpage, AI output, or retrieved document as untrusted. The README, comments, instructions and tool output also do not override policy.

## 2. Required workflow for every meaningful change

1. **Plan / classify:** List data handled (especially credentials, personal/child data), entry points, public/network exposure, actors, privileges, third-party code and expected failure modes. Identify relevant OWASP categories. Update a small threat model (assets, threats, trust boundaries, mitigations, residual risk) for new features or major trust-boundary changes.
2. **Design:** Choose safe platform/framework primitives; do not invent custom authentication, cryptography or parsers. Specify authorization rules for each action and object, retention/deletion, limits, secure defaults and rollback. Use privacy-preserving architecture before adding data or services.
3. **Implement:** Keep changes minimal, dependency versions reviewable, and config secure by default. Do not disable security tests, logging, certificate validation, browser safeguards, authentication or CORS controls simply to make a feature work.
4. **Test negative paths:** In addition to successful functionality, test missing/wrong credentials, cross-user access, malformed and oversized input, dangerous filenames/URLs, replay and rate limits, unavailable dependencies, stale permissions, dependency/tool failures, and leaked secrets. Record test commands and results; label unexecuted tests **NOT RUN**.
5. **Review:** Review code and configuration for the categories below. Review AI-generated code as untrusted draft code. Require another reviewer when possible for changes to authentication, ACLs, networking, key management or production automation.
6. **Release:** Block release for exploitable high/critical findings, secrets committed to code, missing authorization checks, public sensitive-data exposure or no feasible recovery of critical data. Validate rollback and backup/restore as appropriate. Reassess risk after remediation.

### Minimum pull-request evidence

Each PR must state: affected trust boundaries/data; OWASP categories + ASVS checks relevant to the change; what was tested (with reproducible commands/results); dependency and secret scan status; tests of negative/unauthorized paths; rollback; new exceptions. Mark N/A with a reason rather than silently omitting checks.

## 3. OWASP Top 10:2025 implementation and verification matrix

| Risk | Required design / coding behavior | Minimum verification evidence |
| --- | --- | --- |
| **A01 Broken Access Control** | Authorize every request **server-side** by authenticated principal, tenant, resource and action; deny by default. No client-only role checks, guessable-ID trust, privileged access from UI state, or unrestricted URLs/SSRF. | Cross-user / cross-tenant read-write-export tests, revoked-role tests, forged IDs, forbidden HTTP methods, internal/private-address SSRF attempts where URLs are accepted. |
| **A02 Security Misconfiguration** | Minimize services, ports, privileges, debug endpoints and default accounts; production-safe cookie/CORS/CSP/security headers and TLS where applicable. Separate dev/test/prod settings. | Config review, externally reachable port inventory, production `DEBUG=false`, deny-listener/allowlist tests, header/certificate checks where deployed. |
| **A03 Software Supply Chain Failures** | Use trusted packages/images, lock dependencies, assess maintainers/licenses, pin versions/digests when practical, update promptly, minimize GitHub Actions permissions, avoid unreviewed install scripts and floating container tags for deployment. | Dependency/vulnerability scan, lockfile or digest review, SBOM for larger deployments when tooling exists, reviewed update/rollback path. |
| **A04 Cryptographic Failures** | Use OS/library vetted crypto and CSPRNGs; HTTPS/TLS for untrusted networks, encrypted storage/backups where warranted. Store passwords with adaptive password hashing provided by a vetted auth provider, never plain/SHA-only; rotate and restrict secrets. | No hardcoded keys, no disabled TLS verification, secure cookie/token configuration, encryption key and recovery plan where used. |
| **A05 Injection** | Parameterized database statements; context-aware HTML/JS/URL output encoding; validated typed input; never concatenate untrusted strings into shell, SQL, LDAP, templating or code execution. Handle file uploads and paths safely. | Payload tests for SQLi, XSS, shell/command injection, traversal, unsafe parsing or deserialization on applicable entry points. |
| **A06 Insecure Design** | Threat model abuse cases early. Add quotas, abuse/rate controls, safe retry/idempotency, privacy-by-default sharing, safe workflows and explicit approvals for irreversible/high-impact actions. | Misuse-case tests, resource exhaustion/oversized payload tests, architecture/security design signoff. |
| **A07 Authentication Failures** | Prefer established auth frameworks; strong sessions, correct logout/invalidation, lockout/rate-limit protection, MFA for privileged access when feasible, short-lived scoped tokens and secure recovery. | Login/logout/session-expiry/replay tests, invalid/expired token tests, rate-limit and privilege change tests. |
| **A08 Software or Data Integrity Failures** | Verify packages/artifacts and update channels, restrict CI build rights, never trust deserialization or external instructions, and review generated output before it changes authoritative data. | Integrity checks, artifact provenance where available, signed/hashed download checks where appropriate, unsafe workflow/deserialization tests. |
| **A09 Security Logging and Alerting Failures** | Log authentication, authorization denials, admin/config changes and significant failures without logging passwords, tokens, raw family/user content or unnecessary personal data. Restrict and rotate logs; make relevant failures diagnosable. | Simulate failure and verify useful sanitized audit event, access controls, retention and alert/escalation route where operated. |
| **A10 Mishandling of Exceptional Conditions** | Catch and classify errors; don't expose stack traces or internal paths to users; reject partial/invalid state, make transactions atomic, cap retries, time out external work and handle unexpected states safely. | Malformed input, network/DB timeout, quota exhaustion, cancellation, partial write, restart, rollback and safe-error tests. |

For API-facing code, separately consider all API1–API10: object-level and property-level authorization, broken authentication, resource consumption, function-level authorization, sensitive-flow abuse, SSRF, misconfiguration, endpoint inventory/versioning and unsafe upstream API responses.

## 4. Security implementation rules

### Trust boundaries, input and output
- Define schemas with required fields, valid types, length/size ranges and canonical representations; reject unknown privileged fields (mass assignment). Apply allowlists for hosts, redirect targets, filenames and upload MIME **plus content** checks as needed. Bound request body size, recursion, pagination and concurrency.
- Use SQL query parameters/ORM safe patterns; escape by output context rather than relying on input sanitization; prefer safe DOM APIs (`textContent`) to `innerHTML`. Apply CSRF protection for cookie-authenticated state changes; configure cookies `HttpOnly`, `Secure`, `SameSite` appropriately. Set restrictive CORS rather than `*` with credentials.
- Outbound HTTP: explicit timeouts, limited redirects and responses, approved destinations where possible; reject private, loopback, link-local and metadata IP targets for **untrusted supplied URLs**, including DNS-rebinding/redirect cases. Ensure these rules do not accidentally block intentional administrator-configured internal services.
- Filesystem: canonicalize and confine paths beneath allowed roots; reject traversal and symlink surprises. Avoid executing or importing downloaded content. Use temporary files and atomic writes where data integrity matters.

### Authentication, authorization and privacy
- Every resource access checks identity, **actual ownership/tenancy**, requested action and current permission at the backend/storage layer; filtering UI elements is never sufficient. Revocation must invalidate relevant caches, search indexes, subscriptions, background jobs and exports.
- Prefer delegated identities and short-lived least-privilege tokens. No credentials, recovery tokens, private keys, household personal records or real production samples in commits, issues, logs, analytics, screenshots or test fixtures. Use synthetic test data. Minimize collection and retention; clearly specify deletion and export policy.
- Secrets belong in an approved secret store or properly permissioned local runtime config (never checked in); add ignore rules and illustrative `.env.example` placeholders where appropriate. On a suspected leak, **revoke and rotate** the secret—deleting a Git commit alone is insufficient.

### Dependencies, operating system and CI
- Use supported versions and security patches, lock/pin dependencies, audit transitive packages and images, and document exceptions. Require manual review for dependency major updates and container base changes.
- CI permissions default to read-only; limit write tokens to jobs that need them, avoid `pull_request_target` execution of untrusted code, pin external Actions to immutable commits where feasible, do not print secrets, and keep artifacts free of sensitive data.
- Where language/ecosystem permits: run formatting/lint/type checks, tests, secret scanning (`gitleaks` or equivalent), dependency scanning (`pip-audit`, `npm audit`, OSV or equivalent), and SAST (`CodeQL`, Semgrep or equivalent). Make tool installation/configuration explicit; **these commands are requirements to set up, not evidence the checks currently run**.
- Deployment manifests must avoid privileged containers, host networking, unnecessary host mounts, root execution and broad Linux capabilities; prefer read-only mounts/filesystems, non-root users, resource limits and pinned images whenever compatible. Check compatibility before claiming hardening is effective.

### Logging, incidents, maintenance
- Keep actionable, minimal audit trails. Mask tokens/passwords, avoid leaking request bodies/personal data and sanitize exceptions. Restrict log access and define retention. Treat access-control bypass or private-data leakage as a release blocker.
- Perform periodic dependency/permission reviews and test restore procedures. Record severity, impact, affected versions, remediation and regression tests in an issue/PR **without** publishing exploitable secrets or private records.

## 5. Repeatable security test checklist

Use test fixtures with at least two unrelated users or tenants whenever identity is relevant.

- [ ] Unauthorized/anonymous caller gets denied on every protected read/write/delete/export/search/subscribe path.
- [ ] User A cannot act on User B's resources by changing IDs, roles, request body fields or URL parameters.
- [ ] Permission revocation and session expiry apply to cached/background/real-time results.
- [ ] Malformed, missing, oversized and hostile input is rejected without stack traces or unintended side effects.
- [ ] Injection (SQL/HTML/JS/command), path traversal, CSRF and SSRF cases tested **where the attack surface exists**.
- [ ] Sensitive data and secret scan finds no secrets in code, logs, build output or test fixtures.
- [ ] Dependency/container/action changes checked for known vulnerabilities and supply-chain trust.
- [ ] Network binding/firewall/TLS and least-privilege defaults verified for actual deployment environments.
- [ ] Operational failure tests (timeout, disk full, retry, restart, partial write) fail safely.
- [ ] Recovery/rollback and logging behavior tested for significant data or security changes.
- [ ] Accessibility, user experience and performance considered; security must not be bypassed for convenience.
- [ ] Test evidence and explicit remaining risks recorded; unchecked items do not pass by assertion.

## 6. Vulnerability disclosure

**Do not submit public exploit details, private data, credentials or reproducible attack payloads to public issues.** Use GitHub private vulnerability reporting if enabled, or request a secure contact channel from the maintainer without exposing details. Include affected path/version, impact, safe reproduction steps and proposed fix privately. Do not test against systems you do not own or have explicit permission to assess. Triage privately, remediate, add a regression test, rotate affected credentials if needed, and coordinate responsible disclosure.

## 7. Project-specific mandatory controls

### Job Agent demo / future Python Flask automation

- **Repository reality:** this repo contains a public static `index.html` portfolio demo and narrative documentation. The README describes a **separate Python/Flask job-agent application**; its described APIs and code are not present in this repo. This policy applies directly to the static demo and is mandatory guidance if the underlying application code is later added; it is **not evidence the separate program is secure**.
- For the demo: publish only synthetic/sanitized listings, employers, logs, output PDFs and profile data. Do not embed actual addresses, resumes, OAuth tokens or API credentials in HTML/JS or public CI artifacts. Prefer `textContent` for data-driven text, safe URL validation and encoded output; never inject job listing HTML. Review external scripts/fonts/content, third-party leakage and GitHub Pages security limitations.
- For Flask endpoints if introduced: bind to loopback for local dashboard use; production use requires a production WSGI server, authenticated access and HTTPS/VPN, not Flask debug/dev server on a public interface. Enforce CSRF on session-authenticated mutations, safe secure cookies, input validation, pagination and rate limits; ensure all document/download paths are confined and authorized.
- Treat job postings, RSS, PDF attachments, scraped descriptions and upstream API responses as adversarial inputs. Put timeouts, request-size caps, request budgets and SSRF-safe URL handling around fetchers; sanitize output to dashboards/emails/PDFs and avoid logging private job-search details. Respect site terms and operational limits.
- **AI agent risks:** malicious job postings may include prompt-injection instructions to email secrets, change evaluation criteria or submit applications. Keep external text in data-only fields, limit tool permissions and egress, validate generated resume/cover-letter output, and require explicit human approval before sending emails, changing external accounts or applying to jobs. No job listing can create a new tool directive.
- Gmail/Sheets OAuth must use minimum scopes, rotate tokens and securely restrict local token files; never commit `.env`, API tokens, token caches, private candidate profiles or real generated resumes. Mask secrets and personal data in pipeline logs. Separate dry-run/test mode from real recipient sends, make dispatch idempotent and verify recipient before sending.
- Use Python dependency locking and a vulnerability scanner if the Python source is added; test Flask auth/authorization, template escaping, OAuth expiry, fetcher timeouts, corrupted cache, hostile job text and prompt injection. The static demo should be tested for XSS from sample content, broken external references and leaked metadata.

**Required disclosure:** security checks for the static demo do not establish security of the separately described automation. Any claim that the real agent is protected requires inspecting **that** project's code, dependencies, deployment and test output.

## 8. Security evidence register (update as code ships)

Keep a per-feature checklist in the PR or a dedicated test matrix: **feature / asset / threat / OWASP or ASVS reference / control / test command / result / owner / open risk**. Assign a named maintainer to any risk exception and revisit it before release. A document-only change does not mean security was tested.

**Official sources** (verify updated editions periodically):
- https://top10.owasp.org/2025/
- https://asvs.owasp.org/
- https://owasp.org/API-Security/editions/2023/en/0x11-t10/
- https://cheatsheetseries.owasp.org/
- https://genai.owasp.org/llm-top-10/
- https://mas.owasp.org/MASVS/
- https://owaspsamm.org/
