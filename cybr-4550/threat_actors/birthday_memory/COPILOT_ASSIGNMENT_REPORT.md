# Birthday Memory Security and API Testing Report

## 1. Environment information

- Repository: `bboy330/cyber-uvu-student-projects`
- Project: `cybr-4550/threat_actors/birthday_memory`
- Branch: `security-assessment-redteam`
- OS: Windows
- Node.js: v26.13.1
- npm: 11.20.0
- Docker: 29.9.3
- Docker Compose: v5.7.0
- Frontend: React 18 and Vite
- Backend: Express 4
- Database: PostgreSQL 16 Alpine
- API: `http://localhost:4000`
- Frontend: `http://localhost:5173`
- Database host port: `5544`

## 2. Commands executed

- `npm run install:all`
- `docker info --format '{{json .ServerVersion}}'`
- `docker compose config`
- `npm run db:up`
- `docker compose ps`
- `docker port thebirthdates-db`
- `npm --prefix server run seed`
- `npm run dev`
- `curl.exe -i http://localhost:4000/api/health`
- `curl.exe -I http://localhost:5173`
- CRUD, validation, CORS, SQL injection, and XSS `curl.exe` commands
- `docker compose stop postgres`
- `curl.exe -i http://localhost:4000/api/health`
- `docker compose start postgres`
- `docker exec thebirthdates-db psql ...`
- `Test-NetConnection -ComputerName 127.0.0.1 -Port 5544`
- `npm audit --omit=dev --json`
- `npm --prefix server audit --omit=dev --json`
- `npm --prefix client audit --omit=dev --json`
- `docker inspect thebirthdates-db`
- `docker image inspect postgres:16-alpine`

## 3. Problems encountered

- Initial setup succeeded: Docker was running, PostgreSQL became healthy, the API listened on port 4000, and Vite listened on port 5173.
- Dependency installation reported server vulnerabilities; detailed JSON evidence was captured.
- Malformed JSON is logged with a local stack trace and returned as a generic HTTP 500 externally.
- The delete cleanup command initially had a PowerShell piping syntax error; it was corrected and test records were verified absent.
- The initial PostgreSQL PID-inspection command used an option unsupported by BusyBox. A follow-up `/proc/1/status` check confirmed PostgreSQL PID 1 runs as UID/GID 70, not root.

## 4. Troubleshooting performed

- Verified the Git branch and worktree.
- Verified `.env.example` is tracked and `server/.env` is excluded by `.gitignore`.
- Verified Docker information and resolved Compose configuration.
- Started PostgreSQL and confirmed the container is healthy.
- Started the API and frontend development servers.
- Stopped PostgreSQL, verified health endpoint disclosure, and restarted the container.
- Inspected database role flags and privileges directly with `psql`.
- Used `Test-NetConnection` to verify host port reachability.
- Checked for available Trivy, Syft, Grype, gitleaks, and Snyk scanners.

## 5. Security tests performed

- Baseline health and list endpoints
- Unauthenticated GET, POST, PUT, and DELETE
- Invalid field and malformed JSON inputs
- Month and upcoming boundary inputs
- Untrusted Origin and CORS preflight
- Harmless SQL-injection search text
- Benign `<b>` and `<script>` storage probe
- Database outage health disclosure
- Database host reachability and role privilege
- PostgreSQL TLS configuration review
- API HTTPS availability
- Security response headers
- Dependency and container metadata
- Static dangerous-HTML and server-fetch review

## 6. Findings

Eleven confirmed or controlled-risk items are recorded in `security/findings.md`.

The highest-confidence findings are:

1. Broken access control across every CRUD route.
2. PostgreSQL application role is a superuser.
3. Database port is published to host interfaces.
4. CORS is unrestricted.
5. Database TLS certificate verification is disabled.
6. One critical, one high, and one moderate server dependency advisory.
7. Health endpoint discloses database outage text.
8. No CRUD audit trail exists.
9. No security-header baseline or request throttling is present.

## 7. Evidence

- Existing baseline evidence remains under `security/baseline/`.
- Raw API, CORS, validation, injection, database, and container evidence is under `security/evidence/`.
- Scanner JSON and derived summaries are under `security/` and `security/evidence/`.
- Findings and negative results are in `security/findings.md` and `security/not-vulnerable.md`.

## 8. Changes made

- No application or security remediation changes were made for the Track A assessment.
- Safe test records were created and deleted; post-cleanup search returned no test records.
- This report and evidence findings are documentation-only changes.

## 9. Retest results

- Database was restarted and returned to healthy status.
- PostgreSQL health and API endpoint verification succeeded after restart.
- Test cleanup was verified with a search that returned an empty array.
- The frontend production build completed successfully.
- A local browser screenshot was captured at `security/evidence/application-ui-evidence.png`.
- Remediation retesting was not applicable because Track A is assessment-only.

## 10. Lessons learned

- Suspicious code patterns require runtime evidence before being labeled vulnerable.
- Parameterized SQL and React text rendering prevented the tested injection payloads.
- Development convenience settings can become severe findings when credentials and privileges are reused or ports become externally reachable.
- Raw response headers and database-role checks provide stronger evidence than source inspection alone.
- A source-only suspicion should remain a candidate until runtime behavior and code location both support it.

## 11. Final requirement review

| Assignment requirement | What was verified | Evidence / command | Final status |
|---|---|---|---|
| Inspect repository structure | Frontend, API, database, Docker, env, security files, and assignment docs were identified | Repository and source review | Complete |
| Determine run and install commands | npm scripts and Docker startup were confirmed; dependencies installed | `npm run install:all`, `npm run db:up`, `npm run dev` | Complete |
| Capture environment and baseline | Branch, tool versions, worktree, baseline and Docker state were recorded | Git/Docker/Node/npm commands; `security/baseline/` | Complete |
| API verification | Health and CRUD endpoints were exercised | `curl.exe -i` transcripts under `security/evidence/` | Complete |
| Input validation | Missing fields, future date, invalid formats, malformed JSON, month and days boundaries | `validation-invalid-before.txt`, `malformed-json-before.txt`, `validation-month-zero.txt`, `availability-upcoming-days.txt` | Complete |
| Authentication and authorization | All CRUD verbs were sent without Authorization | `api-get-no-auth.txt`, `api-post-before.txt`, `api-put-before.txt`, `api-delete-before.txt` | Complete |
| CORS | Arbitrary Origin and preflight behavior were checked | `cors-untrusted-origin.txt`, `cors-preflight.txt` | Complete |
| SQL injection | Harmless quote/OR payload was used against search | `curl -G ... --data-urlencode "q=' OR '1'='1"`; `sql-injection-safe.txt` | Complete, negative result |
| Script injection / XSS | Benign markup was created and React dangerous-API patterns were reviewed | `xss-storage-before.txt`; source grep; loaded UI screenshot | Complete for API/source/storage, browser execution not dynamically automated |
| Error disclosure | Malformed JSON and database outage were tested | `malformed-json-before.txt`, `error-disclosure-health-outage.txt`; Express server log | Complete |
| Database exposure | Port, host listener, role, superuser flags, privileges, and PII access were checked | `docker compose ps`, `docker port`, `Test-NetConnection`, `psql`, `database-*` evidence | Complete |
| TLS/HTTPS | Plain HTTP API and disabled PGSSL certificate verification were reviewed | `curl -k https://localhost:4000/...` and `server/src/db.js:17` | Complete for local plain-HTTP and static setting; no managed certificate test |
| Security headers and config | Response headers, CORS, container runtime, ports, and logs were checked | `security-headers.txt`, `container-runtime.txt`, `postgres-pid1-user.txt` | Complete |
| Dependency and image review | npm audits and image metadata were captured | `npm audit` JSON and `docker image inspect` | Complete within available tools; no Trivy/Syft/Grype/gitleaks/Snyk installed |
| Container CVE scan | No container scanner was available | Tool availability check; Docker image digest inspection | Incomplete |
| Secret sweep | Basic tracked-file/private-key pattern review found no real secret pattern; `server/.env` is ignored and untracked | `git ls-files`, source regex search, `server/.env` excluded | Complete for basic working-tree review; full git-history scanner not run |
| Remediation | Track A is assessment-only, so no fixes were applied | No application file changes made by this work | Not applicable, as required by selected track |
| Retesting after remediation | No remediation was implemented | No before/after remediation test | Not applicable, as required by selected track |
| Evidence bundle | Raw transcripts, JSON audits, screenshots, and findings registers are present | `security/evidence/`, `security/results/`, `security/findings.*`, `security/not-vulnerable.md` | Complete |
| Board report PDF | Markdown board report exists; no PDF conversion tool or HTML conversion was completed | `security/BOARD_REPORT.md` | Incomplete |
| Public fork push | Repository branch exists locally but was not pushed or made public | Git branch check | Incomplete; external GitHub action was not requested |
| Portfolio root README | Project README exists and security-specific hiring-manager README exists under `security/` | `README.md`, `security/README.md` | Partially complete; a distinct root portfolio README was not created |
| AI usage disclosure | This report documents Copilot/AI-assisted commands and validation; a formal appendix is not yet present | This section and report history | Incomplete until a named appendix is added |
| Oral defense | Scheduling is outside repository work | Canvas submission checklist | Incomplete / user action |

## 12. Final finding review

| Finding | Behavior tested | Evidence supports finding | Remediation tested | Retest |
|---|---|---|---|---|
| BM-01 unauthenticated CRUD | Yes: GET/POST/PUT/DELETE without Authorization | Yes | No; Track A | Not applicable |
| BM-02 PostgreSQL superuser role | Yes: role flags and privileges queried with psql | Yes | No; Track A | Not applicable |
| BM-03 PostgreSQL host port exposure | Yes: Compose mapping and loopback TCP listener | Yes for loopback/all-interface listener; external reachability not tested | No; Track A | Not applicable |
| BM-04 unrestricted CORS | Yes: Origin and OPTIONS preflight | Yes | No; Track A | Not applicable |
| BM-05 disabled PGSSL verification | Static code setting confirmed; local HTTP TLS test confirmed plain HTTP | Yes for source behavior; managed TLS behavior was not dynamically tested | No; Track A | Not applicable |
| BM-06 missing security-header baseline | Yes: raw API response headers | Yes | No; Track A | Not applicable |
| BM-07 health error disclosure | Yes: database stopped and health checked | Yes | No; Track A | Not applicable |
| BM-08 missing CRUD audit trail | Yes: DELETE succeeded and source review found no actor/audit write | Yes | No; Track A | Not applicable |
| BM-10 server dependency vulnerabilities | Yes: npm audit JSON | Yes | No fix applied; npm audit fix was not run | Not applicable |
| BM-11 unbounded list/rate-limit risk | Code and small live GET verified; no load test | Yes for missing design controls, not a demonstrated DoS | No; Track A | Not applicable |

The root-container claim was corrected:

| Retested result | Evidence |
|---|---|
| PostgreSQL PID 1 runs as UID 70, not root | `security/evidence/postgres-pid1-user.txt` |

## 13. AI usage disclosure

GitHub Copilot CLI was used to inspect the repository, execute localhost commands, collect evidence, compare source and runtime behavior, and draft report text. All confirmed findings were checked against raw output or source locations. The selected Track A scope did not authorize remediation changes, so no remediation retest is claimed.

## 14. Updated final limitations

- No PDF board report was generated.
- No public fork push or Canvas submission was performed.
- No external repository, network, or machine was tested.
- No sustained availability test or external database TLS test was performed.
- No container CVE scanner, secret-history scanner, Docker Bench, or SBOM generator was available.
- The visual screenshot shows the loaded UI, but the XSS proof was not automated in a browser.

## 15. Final secret and PII push review

- Working-tree status showed only the assignment report and `security/` assessment artifacts as untracked. No files were staged.
- `server/.env` is ignored by `.gitignore`, is not tracked, and contains only the project's documented local defaults/comments. It must not be added with `git add -f`.
- `git ls-files` shows only `.env.example` files as tracked environment files.
- Separate searches found no private-key PEM headers or common AWS/GitHub/Slack/Google token patterns in the project working tree.
- The same high-confidence token/key-pattern search across the five project-path commits in local Git history found no matches. This is not a full secret scan: `gitleaks` and `trufflehog` are not installed, and a pattern scan can miss novel or encoded secrets.
- The evidence includes the application's shipped sample fixture records (recognizable public names, fabricated/demonstration birthday data, `example.com` addresses, and 555 phone numbers) and the test UUIDs. No personal records supplied by the user were found. If the requirement is to avoid all names that could be interpreted as personal data, redact the fixture rows from raw transcripts/screenshots before a public push.
- The existing tracked Compose configuration and `.env.example` contain predictable development database credentials. They are not credentials supplied by the user, but they are intentionally insecure defaults and are already documented as a security finding. Do not reuse them outside isolated local testing.
- The local app `.env` must remain untracked. Before staging, review the exact path list and staged diff; do not use `git add -A` or `git add .` blindly.
