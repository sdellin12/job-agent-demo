# Coding agent instructions

This file is binding guidance for Codex and all AI-assisted code changes in this repository.

**Read [SECURITY.md](SECURITY.md) before implementing any change.** Apply its OWASP Top 10:2025, ASVS 5.0 and applicable API, mobile or LLM security requirements; do not treat checklists as proof of security. The security requirements apply to code, workflow files, scripts, dependencies, infrastructure manifests, prompts and tests.

## Project-specific guardrails

This repository is a public static showcase. The described Python/Flask job-agent source lives elsewhere and is not automatically available here. Never claim to have audited that agent. Keep sample data synthetic, avoid unsafe DOM APIs and credential exposure, and treat job listings, external links and LLM output as untrusted.

## Required work procedure

1. Read README, SECURITY.md and relevant design/operations docs. Scope the proposed change; list data, actors, trust boundaries, exposures and OWASP/ASVS risks.
2. Prefer the simplest secure-by-default implementation and explicit least-privilege authorization. Preserve existing behavior, accessibility and owner-approved scope; never weaken safeguards for convenience.
3. Implement unit/integration and negative security tests for every new sensitive data path, input/parser, auth rule or networking change. Test unauthorized users, adversarial input, timeouts, error states and rollback as relevant.
4. Run available tests and security tooling (secret scan, dependency scan, SAST and config checks). If tooling is missing, state **NOT RUN** and propose the minimum setup; never invent results or mark security checks passed without evidence.
5. In the final change summary report OWASP/ASVS control mapping, affected files, exact tests executed/results, remaining risks, compatibility issues and any unverified deployment assumptions. Block release for high/critical exploitable findings or data exposure.

No real passwords, OAuth tokens, private keys, personal records, production user data or identifying child information in source, fixtures, screenshots or logs. Use synthetic placeholders. Report vulnerabilities privately.
