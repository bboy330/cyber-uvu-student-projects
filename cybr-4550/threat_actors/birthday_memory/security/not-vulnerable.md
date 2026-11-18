# Tested - Not Vulnerable

## SQL injection

**TEST**

`GET /api/birthdays?q=' OR '1'='1`

**RESULT**

The harmless payload returned HTTP 200 with an empty result set, not altered or exposed records.

**CODE REASON**

`server/src/routes.js` uses positional PostgreSQL parameters (`params`, `$1`, `$2`) for search input. The literal quote payload is used as search text.

**VERDICT**

Not Vulnerable (tested). Parameterized search safely prevents this payload from changing query semantics.

**EVIDENCE**

- `security/evidence/sql-injection-safe.txt`

## Script injection / stored XSS

**TEST**

Created a record containing `<b>Test</b>` and `<script>test()</script>` using test data.

**RESULT**

The API stored and returned the literal strings; no execution response was observed.

**CODE REASON**

React source uses text-bound values and no dangerous HTML rendering API was found.

**VERDICT**

Not Vulnerable (tested) for this local stored-XSS probe.

**EVIDENCE**

- `security/evidence/xss-storage-before.txt`
- Static searches found no `dangerouslySetInnerHTML`, `innerHTML`, `eval`, or `new Function`.

**LIMITATION**

No browser screenshot was captured in this phase. The React source plus API result supports the negative result, but a browser-level visual check should still be added to the final evidence bundle.

## Server-side input validation

**TEST**

POST with missing names, future date, invalid phone, invalid email, and malformed JSON.

**RESULT**

The invalid field payload returned HTTP 422 with field-level errors. Malformed JSON returned HTTP 500 from the generic error middleware.

**VERDICT**

Input validation is present and correctly rejects invalid domain fields. Malformed JSON handling is not disclosure-vulnerable but should be improved from 500 to 400.

**EVIDENCE**

- `security/evidence/validation-invalid-before.txt`
- `security/evidence/malformed-json-before.txt`

## SSRF

**VERDICT**

Not Applicable / No Fetch Surface. The server exposes CRUD and health routes and has no URL-fetch, webhook, image-fetch, or remote-resource parameter.

**EVIDENCE**

- Static search found only the browser API `fetch` call.
- No server-side URL fetching implementation exists in `server/src`.

## PostgreSQL container runtime user

**TEST**

Inspected `/proc/1/status` inside the running PostgreSQL container.

**RESULT**

PID 1 was `postgres` with UID/GID 70.

**VERDICT**

Not Vulnerable (tested) for the specific claim that PostgreSQL PID 1 runs as root. The container's Docker `Config.User` is empty, but the official image entrypoint starts PID 1 as UID 70 in this runtime.

**EVIDENCE**

- `security/evidence/postgres-pid1-user.txt`
