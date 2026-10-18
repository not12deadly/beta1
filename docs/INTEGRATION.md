# Formline 2.0 — Live integration contract
Status: architecture and data-integrity gate documented; **not connected to production**.

## Verified current state (2026-10-18)
- Lovable database has 118 rows in `formline_prediction_imports`, 3,478 `races`, 36,521 `results`, 441 `meetings`.
- Imported `race_key` uses `YYYY-MM-DD:Venue:Rn` (example `2026-10-15:Hawkesbury:R1`).
- Direct import-to-race joins currently returned 0 matched rows. Do not report hit rates until the mapping is reconciled and reviewed.
- The GitHub Pages host is static and public. The existing Lovable admin pages are authenticated and separate.

## Integration design
1. **Private backend only**: create an authenticated API/Edge Function in the existing data service, with read-only scoped endpoints for predictions, races, results and aggregate metrics. Never embed service-role tokens or unrestricted database credentials in public GitHub Pages HTML/JS.
2. **Canonical race identity**: resolve `race_key` to an official race ID by normalized local date, venue aliases and race number, then validate against jump time and source. Ambiguous or unmatched records remain explicitly unmatched; never fuzzy-auto-link two plausible races.
3. **Result provenance**: require source URL/provider, official timestamp, finishing order, scratches and integrity checks. Quarantine synthetic outcomes. A race with no verified results displays `Pending verification`.
4. **Immutable snapshots**: never update imported forecast rows; append new prediction versions with their own `frozen_at`, `input_cutoff` and provenance. Only pre-jump forecasts are eligible for blind-test scores.
5. **Metrics**: separately calculate winner, BOTH horses Top 4, ALL THREE Top 5, four-horse Top 4 and Top 5 coverage. Show numerator, denominator, model, date range, and unresolved races; no invented confidence percentages.
6. **Scheduled ingestion**: backend server-side nightly official results ingestion, idempotent upsert by official race and runner IDs, retry/backfill, audit trail and alert on provider outages. Existing ChatGPT reminder is not a backend cron integration.
7. **Authentication**: authenticated prediction access, admin role checks for imports, row-level permissions, CORS restricted to intended frontend origins, no private data in public static files.
8. **Frontend**: dashboard routes on GitHub Pages call the authorized API; show loading/error/last-refreshed/source statuses. Optimize mobile navigation and avoid full-page fetches; paginate or filter race and meeting queries.
9. **Validation**: test all 30 imported races, multiple venue aliases, timezone edges, scratches, ties/dead-heats, duplicate results, incomplete fields, and no result leakage. Publish only after reconciled samples pass.

## Release gate
Do not label this site 'fully integrated' until the backend API, authentication, results matching, scheduled ingestion and production smoke tests all pass. Public GitHub Pages alone cannot securely replace the private database/API.
