# InventoryOS

A self-hosted inventory application for one organization, built with Next.js, React, TypeScript, and SQLite. Accounts and inventory are stored on the server, shared across signed-in devices. No business inventory is stored in browser localStorage.

## Run locally

Requires Node.js 24 or newer.

```sh
npm ci
npm run setup:local
npm run dev
```

Open http://localhost:3000. Use the installation setup token in `.env.local` to create your first Owner account. Choose your company name and currency. There are no default passwords or public account registration. Initial data includes a main warehouse, a General category, and standard units. Products, contacts, orders, and stock begin empty.

Keep the setup token private. It works only before the first account is created. Existing browser-only inventory is never imported silently: download an old backup and restore it explicitly from the Owner's profile. Inventory restores preserve the installation's login accounts.

## Inventory workflows

- Products: create/edit/archive, categories, units, variants, images, SKU registry, Code 128 labels, keyboard-mode scanner lookup, atomic CSV import, filtered exports.
- Inventory: on-hand/available/reserved/incoming stock, reasoned adjustments and approvals, staged warehouse transfers, physical counts with intervening-movement detection, ledger history, reorder suggestions.
- Purchases: multi-line orders, approval/placement, partial/full receiving, returns against received quantities, history.
- Sales: confirmation/reservations, picking/packing, partial/full dispatch, recorded carrier/tracking references, returns against dispatched quantities.
- Warehouses: stock overview, locations/bin directory, activity. Quantities are tracked at warehouse level.
- Contacts: supplier/customer profiles, terms, order histories, supplier summaries, customer groups.
- Reporting: valuation, movements, low stock, recency, product/warehouse performance, purchasing, sales, adjustments, search/sort/pagination/export.
- Administration: company profile, real login accounts, server-enforced roles, password changes/reset by Owner, account deactivation, approval rules, estimated tax, in-app alerts, business audit trail.

Supported workspace currencies: USD, BDT, EUR, GBP, INR, AED, CAD, AUD, JPY, SGD, PKR, SAR. Currency is selected during setup; existing amounts are never silently relabeled or converted.

## Data and security

All inventory mutations run through authenticated `/api/workspace` commands. The server executes the shared domain rules using the account's current role. Client role values cannot grant access. SQLite `BEGIN IMMEDIATE` transactions, revision checks, and idempotency keys protect writes against conflicts and retried requests. Stock cannot consume reserved units; receiving/dispatch and returns cannot exceed eligible quantities.

Passwords use salted scrypt. The server stores session token hashes; cookies are HTTP-only, SameSite Strict, and Secure in production. Sessions expire after 12 hours and are revoked after password/account changes. Login attempts are rate limited. Mutation endpoints validate the exact request origin. Passwords never appear in workspace payloads or inventory backups.

The SQLite file defaults to `data/inventoryos.sqlite`. It uses WAL and full synchronous durability. An independent server audit table retains account/action/revision records even when an Owner restores an inventory backup. Browser JSON backups include inventory and team profile records, but not credentials or sessions. Full SQLite backups include authentication state and must be kept private.

## Production deployment

See [deployment instructions](docs/deployment.md). A persistent Linux server with Docker, a domain, and HTTPS is the supplied deployment target. This is a single-organization installation, not a multi-tenant public SaaS. Do not deploy SQLite on an ephemeral filesystem, network filesystem, or as multiple independent replicas.

```sh
npm run verify
```

Docker/Caddy configuration provides the app, persistent volumes, health checks, and HTTPS reverse proxy. Automated backup scheduling and off-server retention must be configured by the hosting administrator.

## Verification

```sh
npm run typecheck
npm test
npm run check:design
npm run build
npm run check:routes  # requires a running server
```

Tests cover inventory invariants, partial processing, reservations, transfers, returns, stale counts, roles, CSV atomicity, backup validation, page/dialog rendering, account lifecycle, session revocation, last-owner safeguards, credential rollback, disk persistence, multi-connection revision conflicts, and idempotent retries.

`scripts/smoke-live.mjs` exercises real HTTP APIs against a fresh isolated localhost test installation. It refuses an already configured installation. See deployment instructions for execution.

## Explicit limits

No email/SMS provider, carrier API, external webhook delivery, subscription checkout, or payment collection is connected. Their pages describe this status rather than pretending to perform an external action. Password recovery is performed by an Owner or installation administrator. In-app alerts and manually entered tracking references work.

Valuation uses current recorded unit costs. Receipts do not automatically recalculate costs. Reports show stock recency, not FIFO lot aging. Tax and margins are estimates. This release does not implement FIFO/weighted-average costing, lot/expiry/serial tracking, kit assembly, or bin-level quantities. Barcode printer/scanner hardware must be verified with actual equipment before operational rollout.

Full browser interaction, accessibility, mobile-device, load, and hosting recovery checks remain release gates. Passing code tests and a production build does not verify those environments.

Barcode encoding uses [JsBarcode](https://github.com/lindell/JsBarcode).

# inventoryOS-SaaS-System
