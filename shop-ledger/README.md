# Ledgerleaf Shop Book (Phase 1 prototype)

A calm, mobile-first, offline-first money and stock book for small shops. This app lives in shop-ledger/ so it does not overwrite the existing Elysium Dubs Hub project.

## Run locally
Requires Node.js 20+.

    cd shop-ledger
    npm install
    npm run dev
    npm test
    npm run typecheck
    npm run build

The sample provision shop and products are seeded on first open. Change shop name, currency, opening cash and language from Shop settings. English and Yoruba translations are centralised in src/i18n.ts.

## Architecture
- UI: React, TypeScript and Vite; responsive down to 320px with touch-friendly controls.
- Local persistence: Dexie/IndexedDB. Writes are persisted before the UI confirms success.
- Offline shell: service worker caches the app shell. First load needs a connection; after assets are cached, the shell can load offline.
- Money: integer minor units and round-half-up conversion. NGN, GHS, KES, ZAR, USD, GBP and EUR use two decimal places; JPY and KRW use zero. Expand into a maintained currency metadata table before launch.
- IDs and time: device UUID, happenedAt, recordedAt and sourceDevice. Device clock values are not trusted; a future backend must add server receipt time and clock-skew handling.
- Append-only records: sales, expenses, customer payments, stock movements and day closes are new rows. Sale reversals point to the original sale and carry a reason.
- Credit is calculated from sales less payments. Stock is initial quantity plus movements. CSV export includes IDs, timestamps, currency and integer amounts.

## Important limitations
This is a local demo, not a production financial service. Phone OTP, offline PIN unlock, device encryption, server auth/RLS, sync API/outbox, idempotent server writes, multi-device conflict resolution, CSV import, product photos, PDF statements, busy-hour analytics, complete report drill-downs, configurable low-stock thresholds, full payment selection for every flow, and complete day-lock enforcement are not implemented. Browser throttling, Android memory profiling, restart E2E tests and a security review have not been run. The UI explicitly says sync is not connected. Do not use real customer financial data in this prototype or market it as encrypted/production-ready.

## Backend direction
Use PostgreSQL with business-scoped tables and row-level security. Each row should carry business_id, stable client UUID, source_device, happened_at, recorded_at and server received_at. Enforce unique (business_id, id). A device outbox retries until server acknowledgement; server writes must be idempotent. Preserve sync conflicts and resolve them with explicit reversal/adjustment records, never silent overwrites.

## Deployment
Deploy shop-ledger/ as its own Vite project and serve over HTTPS for service-worker support. The existing Elysium Dubs Hub project and deployment configuration are intentionally left untouched.