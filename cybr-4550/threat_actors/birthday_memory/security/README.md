# Security Assessment Portfolio

## Running the application

```powershell
npm run install:all
Copy-Item server\.env.example server\.env
npm run db:up
npm --prefix server run seed
npm run dev
```

The API runs at `http://localhost:4000` and the frontend at `http://localhost:5173`.

## Assessment artifacts

- `findings.md` - structured findings register
- `not-vulnerable.md` - negative test results
- `BOARD_REPORT.md` - board-facing assessment summary
- `evidence/` - raw HTTP, Docker, database, and browser evidence
- `results/` - test identifiers and derived assessment state
- `npm-audit-*.json` - dependency audit results

## Test scope

Testing was restricted to the local application and local PostgreSQL Docker container. Test payload names and addresses used only safe values such as `Copilot Tester`, `test@example.com`, and `localhost`.

## Reproducing the main findings

See `findings.md` for numbered reproduction steps and actual curl output locations.
