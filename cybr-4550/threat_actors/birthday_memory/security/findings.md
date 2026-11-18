# Birthday Memory Findings Register

Methodology: OWASP Web Security Testing Guide mapped to OWASP Top 10 2021.

| ID | Title | OWASP category | Severity | CVSS 3.1 | Status |
|---|---|---|---|---|---|
| BM-01 | All CRUD routes lack authentication and authorization | A01 / A07 | Critical | 9.1 / `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N` | Confirmed Vulnerable |
| BM-02 | PostgreSQL application role is a superuser | A01 / A05 | Critical | 9.8 / `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` | Confirmed Vulnerable |
| BM-03 | PostgreSQL port is published to all host interfaces | A05 | Medium | 6.5 / `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:L/A:N` | Confirmed Vulnerable |
| BM-04 | CORS permits arbitrary origins | A05 | Medium | 5.3 / `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` | Confirmed Vulnerable |
| BM-05 | Database TLS certificate verification is disabled | A02 | High | 7.4 / `CVSS:3.1/AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:N` | Confirmed Vulnerable |
| BM-06 | No security-header baseline | A05 | Low | 5.3 / `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` | Confirmed Vulnerable |
| BM-07 | Database outage details are exposed by health endpoint | A05 | Low | 5.3 / `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` | Confirmed Vulnerable |
| BM-08 | CRUD actions have no audit trail | A09 | Medium | 5.3 / `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` | Confirmed Vulnerable |
| BM-10 | Server dependencies contain one critical, one high, and one moderate advisory | A06 | Critical/High | Multiple; see advisory details | Confirmed Vulnerable |
| BM-11 | Birthday list and search are unbounded and lack rate limiting | A04 / Availability | Low | 5.3 / `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:L` | Confirmed Risk |

## BM-01: All CRUD routes lack authentication and authorization

**LOCATION**

- `server/src/index.js`: middleware registration, no authentication middleware
- `server/src/routes.js`: GET, POST, PUT, and DELETE handlers

**DESCRIPTION**

Every birthday route accepts requests without checking identity or ownership. The source contains no authorization, session, or ownership concept.

**EVIDENCE**

- `security/evidence/api-get-no-auth.txt`: unauthenticated `GET /api/birthdays` returned HTTP 200 and all visible records.
- `security/evidence/api-post-before.txt`: unauthenticated `POST` returned HTTP 201 and created `Copilot Tester`.
- `security/evidence/api-put-before.txt`: unauthenticated `PUT` returned HTTP 200 and changed the test record.
- `security/evidence/api-delete-before.txt`: unauthenticated `DELETE` requests returned HTTP 204.
- `security/evidence/cleanup-verification.txt`: test records were absent after deletion.

**REPRODUCTION**

1. Start the application and database locally.
2. Run `curl -i http://localhost:4000/api/birthdays`.
3. POST a test payload without an Authorization header.
4. PUT and DELETE the returned UUID without an Authorization header.

**CVSS METRIC Rationale**

The network-reachable API requires no privileges or user interaction and permits confidentiality and integrity impact. The vector is `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N`.

**BUSINESS IMPACT**

Anyone who can reach the API can enumerate PII and create, alter, or destroy records. This creates privacy, integrity, operational, breach-notification, and reputational risk.

**RECOMMENDATION**

Add server-side authentication and a role/ownership model. Protect every route with middleware, derive identity from an expiring signed token or secure server session, and authorize record operations against an owner ID. Never rely on the client or UUID secrecy.

## BM-02: PostgreSQL application role is a superuser

**LOCATION**

- `server/.env`: application role `birthday`
- `docker-compose.yml`: role/password provisioning
- `server/src/db.js`: connection pool uses environment or development defaults

**DESCRIPTION**

The application connects as a PostgreSQL role whose flags include `rolsuper=true`, `rolcreaterole=true`, `rolcreatedb=true`, `rolcanlogin=true`, and `rolbypassrls=true`.

**EVIDENCE**

- `security/evidence/database-role-flags.txt`: confirms superuser and other elevated flags.
- `security/evidence/database-role-summary.txt`: identifies `birthday|birthday|thebirthdates`.
- `security/evidence/database-reachability-privilege.txt`: confirms direct role access to schema and PII.

**CVSS METRIC Rationale**

With reachable credentials and a network-exposed database, compromise can produce complete confidentiality, integrity, and availability impact. Vector: `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`.

**BUSINESS IMPACT**

Any database credential or application compromise could grant unrestricted database administration, cross-database access, and destructive power.

**RECOMMENDATION**

Create separate migration and runtime roles. The runtime role should have only CRUD permissions on `birthdays`, no superuser, no role/database creation, and no bypass-RLS capability. Store credentials in deployment secrets.

## BM-03: PostgreSQL port is published to all host interfaces

**LOCATION**

- `docker-compose.yml`: `5544:5432`

**DESCRIPTION**

Docker reports `0.0.0.0:5544` and `[::]:5544`, and `Test-NetConnection` confirmed loopback TCP reachability.

**EVIDENCE**

- `security/evidence/database-host-reachability.txt`
- Docker Compose output in `security/evidence/database-role-summary.txt`

**LIMITATION**

The test proves loopback and host-listener exposure, not reachability from another machine or an external network.

**BUSINESS IMPACT**

On an untrusted or shared network, the database may be reachable by unintended hosts and constrained only by firewall policy.

**RECOMMENDATION**

Bind only to loopback for local development, for example `127.0.0.1:5544:5432`, and use internal Docker networking in deployed environments.

## BM-04: CORS permits arbitrary origins

**LOCATION**

- `server/src/index.js:10`: `app.use(cors())`

**DESCRIPTION**

The API responds to a request with `Origin: http://example.test` with `Access-Control-Allow-Origin: *` and supports POST preflight headers.

**EVIDENCE**

- `security/evidence/cors-untrusted-origin.txt`
- `security/evidence/cors-preflight.txt`

**CVSS METRIC Rationale**

At present the API does not use browser credentials, so cross-origin impact is limited to public data and unauthenticated operations. Vector: `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N`.

**BUSINESS IMPACT**

Arbitrary websites can call the API if it is reachable from the victim browser. Impact becomes more severe when authentication and cookies are later added.

**RECOMMENDATION**

Allow only explicit trusted origins such as `http://localhost:5173` in development and the production frontend origin in production. Reject unknown origins and do not use wildcard CORS with credentials.

## BM-05: Database TLS certificate verification is disabled

**LOCATION**

- `server/src/db.js:17`: `{ rejectUnauthorized: false }`

**DESCRIPTION**

When `PGSSL=true`, PostgreSQL encryption is enabled but the server certificate is not verified.

**EVIDENCE**

- Static source evidence: `server/src/db.js:17`.
- `security/evidence/tls-api-before.txt`: direct HTTPS to Express API failed, confirming plain HTTP in the local runtime.

**BUSINESS IMPACT**

A network attacker able to intercept the database connection could present a fake certificate and read or alter PII despite TLS being enabled.

**RECOMMENDATION**

Use `rejectUnauthorized: true` with a trusted CA and SNI/hostname configuration. Where managed providers supply CA files, load them explicitly and verify the provider certificate.

## BM-06: No security-header baseline

**LOCATION**

- `server/src/index.js`: no Helmet or equivalent middleware

**DESCRIPTION**

Observed API responses lack a Helmet-style baseline including CSP, frame protections, and HSTS. The API also identifies itself with `X-Powered-By: Express`.

**EVIDENCE**

- `security/evidence/security-headers.txt`

**BUSINESS IMPACT**

Browser hardening is weaker. This may enable or amplify clickjacking or script-content risks when deployed as a browser-facing service.

**RECOMMENDATION**

Add Helmet with environment-aware CSP/HSTS/frame-ancestors settings and remove or disable `X-Powered-By`.

## BM-07: Database outage details are exposed by health endpoint

**LOCATION**

- `server/src/index.js:13-19`

**DESCRIPTION**

The health endpoint serializes `error.message` during database failure.

**EVIDENCE**

- `security/evidence/error-disclosure-health-outage.txt`: a stopped database produced HTTP 503 with `"error":"Connection terminated unexpectedly"`.

**BUSINESS IMPACT**

Error text reveals internal database failure state and can assist reconnaissance or monitoring abuse.

**RECOMMENDATION**

Return a generic `{"status":"degraded"}` response to clients while logging sanitized diagnostic detail server-side.

## BM-08: CRUD actions have no audit trail

**LOCATION**

- `server/src/routes.js`: create, update, and delete handlers
- No audit logging middleware or audit table exists in inspected source

**DESCRIPTION**

There is no authenticated actor or persistent audit event for destructive operations.

**EVIDENCE**

- The DELETE probe succeeded without Authorization.
- Source review found no audit log write or actor identity in CRUD handlers.

**BUSINESS IMPACT**

A malicious or accidental delete cannot be attributed to an actor or reliably investigated.

**RECOMMENDATION**

Add identity-aware audit events containing actor, action, record ID, timestamp, outcome, source IP, and minimal change metadata. Do not log PII or credentials unnecessarily.

## BM-10: Server dependency vulnerabilities

**LOCATION**

- `server/package.json`
- `security/npm-audit-server.json`

**DESCRIPTION**

The server dependency audit found:

- `proxy-addr`: critical, CVSS 9.1
- `body-parser`: high, CVSS 7.5
- `qs`: moderate, CVSS 5.3

**EVIDENCE**

- `security/evidence/npm-audit-summary.txt`
- `security/evidence/npm-audit-details.txt`

**RECOMMENDATION**

Upgrade or reinstall affected transitive dependencies to patched ranges, verify lockfile changes, rerun npm audit, and repeat HTTP validation.

## BM-11: Unbounded list operations and no throttling

**LOCATION**

- `server/src/routes.js:47-61` list query
- `server/src/routes.js:68` upcoming all-row query
- No rate-limiter middleware in `server/src/index.js`

**DESCRIPTION**

The list endpoint returns every matching row; the upcoming endpoint loads every row before filtering. No pagination or request throttle was observed.

**EVIDENCE**

- `curl` returned all eight seeded records plus test records in one response.
- `security/evidence/availability-upcoming-days.txt`: a `days=999999` request was capped at 366 by code.

**LIMITATION**

A sustained load test was not run. This finding is a controlled code/condition result, not a demonstrated denial of service.

**RECOMMENDATION**

Add bounded pagination and maximum page size, cap search input length, add caching where appropriate, and add deployment or application rate limiting.
