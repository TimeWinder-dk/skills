---
name: billetto-tw-tickets
version: 1.0.0
releaseDate: 2026-10-25
description: |
  How to replace Place2Book with Billetto inside the tw-tickets repo (SPFx webparts + the TwTicketsProxy Azure
  Function) without breaking the Table Storage contract that Operations Hub reads: a BillettoService behind the
  same shapes as Place2BookService, arrivals derived from attendee scannings (entrance only), ticket types and
  BI report instead of P2B price tiers, an optional signed webhook route, env/bicep wiring, ES5 constraints and
  the route whitelist. Use this skill whenever a task mentions tw-tickets, TwTicketsProxy, TicketsArrivals,
  TicketPrices, syncArrivals, Place2BookService, billetskanning/billetstatus webparts, or moving the SharePoint
  ticket dashboards to Billetto. Read billetto-platform first; coordinate with billetto-ops-hub because Ops Hub
  reads this repo's table directly.
owner: TimeWinder IT
lastReviewed: 2026-10-25
---

# Billetto in tw-tickets

tw-tickets is the only place that talks to the ticket vendor. It syncs vendor data into Azure Table Storage
(`twticketssa`) every five minutes and serves SharePoint webparts (billetskanning, billetstatus, kasseopgoer,
salgsrapport, dataimport) through the TwTicketsProxy function. Operations Hub reads `TicketsArrivals` straight
from that storage account with a managed identity. Swapping Place2Book for Billetto therefore has one rule
above all others: **the table contract stays, the vendor client changes.**

## DO NOT USE FOR

- Zettle sales import (`ZettleService`, `ZettleImportParser`) — unchanged by the vendor swap.
- Website embeds — `billetto-web-sales`.
- Operations Hub wristband rules — `billetto-ops-hub`.

## Required workflow

1. Read `billetto-platform` and its `references/api-reference.md`; then read
   [references/billetto-service-contract.md](references/billetto-service-contract.md) for the field mapping.
2. Read tw-tickets `AI.md` (ES5 lib, Heft is the truth, `WHITELISTED_ROUTES`, the external reader of
   `TicketsArrivals`).
3. Keep `ITicketArrival`, `IEvent`, `ITicketType`, `IProductSale` and the `TicketsArrivals` /
   `Events` / `TicketPrices` tables stable. New vendor fields go in *additional* columns.
4. Implement `BillettoService` with the same three capabilities Place2Book offered (events, arrivals since
   cursor, price/sold counters), then switch the three timers (`syncArrivals`, `syncPrices`,
   `EventSyncService`) to it behind a single `TICKET_VENDOR` switch so a rollback is one app setting.
5. Add the webhook route only after the timer path works; the timer stays as the reconciliation path.
6. Tell Operations Hub (EPIC-21 decision file and this skill's sibling) before changing any column the
   contract lists.

## Mapping at a glance

| tw-tickets concept | Place2Book today | Billetto |
|---|---|---|
| Event list | `GET /event_makers/{id}/events` | `GET /organiser/events` (`name`, `starts_at`, `ends_at`, `state`, `public`, `availability.available`) |
| Arrival | `ticket_arrivals` row (`arrived_at`, `arrived_at_scanner`) | one `scannings.data[]` entry with `mode = entrance` on an attendee (`scannings` is a list object) |
| Arrival id (`rowKey`) | `ticket_arrival_id` | scan log `id` (fallback `attendeeId:scanned_at`) |
| `kasseNavn` | `arrived_at_scanner` | `terminal_name` |
| `ticketId` | `ticket_id` | attendee `id` (UUID) |
| `ticketTypeId` | heuristic from event name | ticket type name (`partout`/`familie`/day) — now real data |
| `numberOfItems` | hard-coded 1 | 1 (packages already yield one attendee per person) |
| Price tiers / sold | `prices` with `sold_tickets`, `door_sales` | `GET /organiser/ticket_types?event={id}` (`quantity`, `price` in øre, `type` incl. `BundleTicketType`) + attendee counts by `state`; the BI report 404s for TimeWinder's events today |
| Incremental cursor | `since=` | `updated_after=` + `__synced` sentinel |

## Non-negotiables

- **Exit scans are not arrivals.** Filter `mode === 'entrance'`; store exits in a separate table if the
  gates want "people on site" numbers.
- **Cancelled/refunded tickets can still carry old scans.** Only count scans of attendees whose current
  `state` is an admission state; re-evaluate on every sync, do not cache validity.
- **Rate limit is cost-based.** Page with `limit=100`, honour `X-Ratelimit-Remaining`, back off on 429 for at
  least a minute, and never fan out per attendee when the list endpoint already includes `scannings`.
- **Pin `Billetto-Version`** in the service; log it in `/diag`.
- **Secrets**: `BILLETTO_API_KEY` as a bicep param → app setting (same shape as `place2bookApiToken`), stored
  in Operations Hub's `tw-opshub-kv` like `PLACE2BOOK-API-TOKEN` so rotation has one home. tw-tickets has
  no vault of its own.
- **ES5**: no `includes`, `flat`, `Object.entries`, `padStart`; follow the existing `indexOf` style.
- **Route table**: any new HTTP route must be added to `WHITELISTED_ROUTES` in `src/index.ts` as
  `METHOD /segment`; the webhook route must bypass Entra token validation and validate `Billetto-Signature`
  instead.

## Webparts

`billetskanning` and `billetstatus` consume `/arrivals` and `/summary`; with the contract intact they need
no code change. Expect two visible differences and handle them in the UI: scanner names change (Billetto
terminal names are free text chosen on the device — publish a naming standard before the festival), and
family tickets now produce several scans where Place2Book produced one (update any "per ticket" wording).
`salgsrapport`'s online rows come from `TicketPrices`; feed that table from Billetto ticket types + sold
counts so year-over-year reports keep working.

## Validation checklist

- [ ] `TicketsArrivals` columns and the `__synced` sentinel unchanged; new columns additive.
- [ ] Only `entrance` scans of attendees in an admission state are stored as arrivals.
- [ ] `ticketTypeId` now carries the real Billetto ticket type name; old heuristic removed or fenced.
- [ ] Vendor switch is one app setting; Place2Book code path still builds until 2026 reports are closed.
- [ ] `/diag` shows vendor, pinned API version, last sync per event and last rate-limit headers.
- [ ] Webhook route (if added) verifies HMAC with constant-time compare, is idempotent on scan id, and is in
      the whitelist; timer reconciliation still runs.
- [ ] Heft build green; tw-tickets `AI.md` and Operations Hub EPIC-21 decision updated in the same change.
