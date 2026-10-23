# GadgetBari — Gadget e-commerce with inventory, accounting & profit-loss

A complete store for the Bangladesh market (BDT ৳, Bangla/English, all 64 districts) with an admin back office for
products, purchases, stock, orders, payments, returns, expenses and real double-entry accounting.

```
Customer store  ──►  Next.js server (actions + API)  ──►  PostgreSQL (single source of truth)
                         │  services/ (business rules, all money maths)
Admin dashboard ──►      │  ledger (double-entry, append-only, idempotent)
                         └─ adapters: bKash · SSLCommerz · SMS · Email · S3 · courier
```

| | |
|---|---|
| Stack | Next.js 15 (App Router) · React 19 · TypeScript · Tailwind CSS 4 · PostgreSQL 16 (`pg`, hand-written SQL migrations) · Zod · Vitest |
| Why no ORM | Accounting needs exact control: `NUMERIC` money, `SELECT … FOR UPDATE`, conditional atomic `UPDATE`s, deferred constraint triggers that reject unbalanced journals. Plain SQL keeps those visible. |
| Money | `NUMERIC(14,2)` in the DB, integer paisa in JS. Never floating point. Unit costs `NUMERIC(14,4)`. |
| Costing | Weighted average per SKU, applied everywhere. |

## Quick start (local)

```bash
cp .env.example .env            # set DATABASE_URL and SESSION_SECRET
npm install
npm run db:migrate              # creates schema
npm run db:seed                 # chart of accounts, roles + demo catalog/orders (dev only!)
npm run dev                     # http://localhost:3000  ·  admin: http://localhost:3000/admin
npm test                        # 41 integration tests against a real PostgreSQL (TEST_DATABASE_URL)
```

Demo staff (password `Demo12345!` — demo data only): `admin@demo.local` (Super Admin), `manager@`, `orders@`, `inventory@`,
`accounts@`, `support@demo.local`. For a real install use `npm run db:seed -- --base-only` and
`npm run admin:create -- --email you@shop.com` (see [docs/ADMIN_SETUP.md](docs/ADMIN_SETUP.md)).

Docs: [Installation](docs/INSTALL.md) · [Deployment](docs/DEPLOYMENT.md) · [Backup & restore](docs/BACKUP.md) ·
[Admin setup](docs/ADMIN_SETUP.md) · [Accounting rules](docs/ACCOUNTING.md) · [API credentials needed](docs/CREDENTIALS.md) ·
[Acceptance checklist](docs/ACCEPTANCE.md)

## What's in it

**Store (customers):** home (featured, best sellers, sale, new, brands, verified reviews, offers, newsletter), category &
search with brand/price/stock filters and 5 sort orders, product page (gallery, variants, specs, warranty, delivery &
return info, Add to cart / Buy now, related, verified reviews, JSON-LD), cart, **guest checkout** (no account needed),
order confirmation with private tracking link, track by order no. + phone, optional account (history, saved addresses,
re-order, wishlist), Bangla/English, dark/light, consent banner gating GA / Meta Pixel / abandoned-cart tracking,
sitemap, robots, canonical & Open Graph.

**Admin:** dashboard (today/month sales, pending, low stock, stock value, COGS, gross & net profit, refunds, chart by
day/week/month), orders (confirm → process → ship → deliver, cancel, failed delivery, COD collection, manual payment
verification, refunds, returns, edit address/courier/notes, invoice, timeline + audit trail), products & variants
(SKU, barcode, price, compare-at, images upload, specs, SEO), categories/sub-categories & brands, inventory
(on-hand/reserved/available/in-transit, adjustments: damaged, lost, count correction, opening stock; full movement
history), purchases (landed-cost allocation, receive, pay, print), suppliers (dues, history, payments, receipts),
customers (guest profiles grouped by phone, PII masked by role), coupons, reviews moderation, returns & warranty,
payments & reconciliation, expenses (vouchers, void by reversal), cashbook (cash/bank/bKash/Nagad/Rocket/clearing
balances, capital, drawings, loans, transfers, VAT paid), 23 reports with print + CSV + Excel, employees & roles with
editable permissions, settings, notifications, append-only audit log.

## Clearly marked limits

- **Payment gateways** (bKash Tokenized, SSLCommerz) and **SMS/email/S3/courier APIs** are implemented against their
  documented endpoints but **untested without real credentials**. Without keys the store runs in an honest demo mode:
  card checkout is disabled with a "not configured" label, bKash/Nagad/Rocket use Send-Money + TrxID verified by staff,
  and messages are logged as `not_configured`. Nothing is ever marked paid without verification. See
  [docs/CREDENTIALS.md](docs/CREDENTIALS.md).
- **Courier API**: manual courier entry (name, consignment ID, tracking no., charge) is complete; automatic booking via a
  courier API is an adapter slot, not implemented.
- **Admin UI language** is English (the storefront is fully bilingual).
- **Order items can't be edited after placing** (by design — cancel and re-order keeps stock and accounts exact).
- **Upazila** is free text with Dhaka-thana suggestions (full 495-upazila list not bundled); delivery zone uses
  district + Dhaka Metropolitan thana.
- **Credit (baki) sales** aren't offered at checkout; the customer-due report shows COD awaiting remittance / advances.
- **Stock transfer between warehouses**: single-location inventory; the movement type exists for a future multi-warehouse module.
- **Purchase returns to supplier**: record via a stock adjustment (lost/damaged) for now.
