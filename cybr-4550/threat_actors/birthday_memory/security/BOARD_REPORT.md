# Birthday Memory Security Assessment

**Classification:** Internal Assessment  
**Version:** 1.0  
**Assessment date:** 2026-11-18  
**Branch:** `security-assessment-redteam`  
**Methodology:** OWASP WSTG, with OWASP Top 10 2021 category mapping

## Executive summary

Birthday Memory should not ship in its current state. The assessment confirmed that any caller able to reach the API can read, create, update, and delete all birthday records without authentication. The PostgreSQL application role also has full administrative database privileges, and the database port is published to host interfaces. The application uses predictable development credentials and does not verify database TLS certificates when PGSSL is enabled.

These conditions expose personal information and allow unauthorized record changes or database-level compromise. Immediate work is required for authentication, least-privilege database access, network exposure, and TLS verification. SQL injection and the tested script-injection payload were not exploitable, and the final report records these negative results.

## Scope and methodology

The in-scope systems were the Express API in `server/src`, React/Vite frontend, PostgreSQL configuration and role, Docker Compose configuration, and local evidence bundle. Testing was limited to localhost and the authorized Docker database.

OWASP Web Security Testing Guide procedures were used with OWASP Top 10 2021 mapping. Evidence was collected with `curl`, Docker and PostgreSQL commands, npm audit, source review, and browser screenshots.

## Key findings

| Rank | Finding | Risk |
|---|---|---|
| 1 | Unauthenticated CRUD | Critical |
| 2 | PostgreSQL superuser application role | Critical |
| 3 | Vulnerable server dependencies | Critical/High |
| 4 | Host-published PostgreSQL | High/Medium |
| 5 | Disabled PostgreSQL certificate verification | High |
| 6 | Unrestricted CORS | Medium |
| 7 | No CRUD audit trail | Medium |

Detailed reproduction steps and raw evidence are in `findings.md`.

## Risk analysis

The combined risk is high. The most likely successful attack is unauthorized access or modification of PII by any local or network participant able to reach the API or database. Administrative database privileges increase the impact of credential or application compromise to full database control.

## Regulatory and compliance considerations

The stored fields include names, birthdates, phone numbers, and email addresses. Depending on residency and deployment, this PII could raise obligations under GDPR, CCPA/CPRA, and state breach-notification laws. The application should maintain an incident-data inventory and breach-response process before public deployment.

## Recommendations and roadmap

**This week**

1. Add authentication and authorization to every CRUD route.
2. Create a least-privilege database role and remove predictable runtime credentials.
3. Bind PostgreSQL to loopback or internal Docker networking.
4. Patch the server dependency advisories.
5. Enable real database certificate verification.

**Next 30 days**

1. Add an explicit CORS allowlist and security headers.
2. Add pagination and rate limiting.
3. Add actor-aware CRUD audit events.
4. Configure container least privilege and resource limits.
5. Add security regression tests.

**Before production**

1. Use HTTPS for browser/API traffic and managed secrets for credentials.
2. Add monitoring and alerting for unauthorized access and destructive operations.
3. Add a deployment SBOM and routine dependency/image scanning.

## Tested but not vulnerable

- SQL injection: parameterized query behavior prevented the harmless payload from changing results.
- Stored script/XSS: React text rendering and absence of dangerous HTML APIs prevented the test markup from executing.
- SSRF: no server-side remote URL or resource-fetch surface exists.

## Limitations

No external network or system was tested. Container CVE scanning was not performed because no container scanner was installed. Browser screenshot evidence was limited to the local UI. The database outage test was brief and non-destructive.

## Conclusion

The application should not be exposed to untrusted networks or used with real PII until the critical access-control and database privilege findings are remediated and retested.
