---
name: billetto-ops-hub
version: 1.0.0
releaseDate: 2026-10-25
description: |
  How Operations Hub (drift hub) consumes Billetto: keep the vendor-neutral ProductSaleRow/scan contract from
  EPIC-21/T-21-01, choose between reading tw-tickets' TicketsArrivals (default) and a direct Billetto adapter or
  signed webhook route, feed wristband accounting with real ticket types instead of multipliers, derive pre-day
  wristband orders from Billetto sales, expose a cached public "tickets left" endpoint, produce the deposit
  refund list, and wire config/Key Vault/route-guard/registry correctly. Use this skill for any Operations Hub
  task that mentions Billetto, armbånd/wristband accounting, ticketArrivalsStore, ticketSalesClient,
  deliveryOverview scans, refillEngine scan gating, billetter solgt/forsalg, depositum, live counters for the
  website, or webhooks from the ticket vendor. Read billetto-platform first; pair with operations-hub-domain,
  integration-resilience and security-hardening.
owner: TimeWinder IT
lastReviewed: 2026-10-25
---

# Billetto in Operations Hub

Operations Hub never sells tickets and never persists sales rows (T-2103). It consumes ticket facts to answer
operational questions: how many wristbands of which colour are left at which entrance, when to order more,
how many guests are pre-sold per day, and which deposit tickets must be refunded. The vendor is hidden behind
one adapter so that the 2027 switch from Place2Book to Billetto changes one file and a config block, not the
calculation.

## DO NOT USE FOR

- The website (`billetto-web-sales`), tw-tickets internals (`billetto-tw-tickets`), scanner apps
  (`billetto-own-ticket-app`).
- Volunteer access or crew wristbands — Volunio domain.

## Required workflow

1. Read `billetto-platform` and `docs/ai/decisions/EPIC-21-VAREGAARD-DELIVERY-SALES.md` plus task T-21-01
   in the Ops Hub repo. The vendor-neutral layer is `shared/src/wristbandAccounting.ts`, the only adapter is
   `backend/src/services/ticketArrivalsStore.ts`, the sales client is `ticketSalesClient.ts`.
2. Pick the integration path with the table below; default is **A**.
3. Read [references/adapter-and-webhook.md](references/adapter-and-webhook.md) for the concrete shapes,
   route-guard/registry rules and config keys.
4. Keep every Billetto-specific string (field names, API version, event ids) inside the adapter or config.
   `salesMapping`, `deliveryOverviewService`, `refillEngine` and the frontend must stay vendor-agnostic.
5. Fail loud: an unreachable or unconfigured source yields `scanSalesAvailable: false` and warnings, never
   "0 sold".
6. Update the EPIC-21 decision file and, when path A is used, note the dependency in tw-tickets `AI.md`.

## Integration paths

| Path | What | When |
|---|---|---|
| **A. Read tw-tickets' table (today's design)** | tw-tickets swaps its vendor; Ops Hub keeps `TicketArrivalsStore` unchanged | Default. Zero Ops Hub code if the table contract holds. One vendor client in the whole organisation. |
| B. Direct `BillettoArrivalsStore` | Ops Hub calls `GET /organiser/events/{id}/attendees` itself and maps `scannings` to `ProductSaleRow` | Only if tw-tickets is retired or lags. Requires the API key in `tw-opshub-kv` and its own rate-limit handling. |
| C. Webhook `billetto/webhook` into Ops Hub | Billetto pushes `attendee.scanned` etc.; Ops Hub invalidates its cache or triggers a refresh | Add-on to A or B for freshness. Must not become a sales store: persist at most "something changed at T", never the rows. |

Whatever the path, `ProductSaleRow` for a scan is: `saleId = scan-<eventId>-<scanId>`, `productName` = the
ticket type name (Billetto gives it; Place2Book only gave the event name), `quantityUnits = 1`,
`soldAt = scanned_at`, `cashRegisterName = terminal_name`, `source = 'scan'`, `refund = false`.

## Wristband accounting with real ticket types

- Rules with `source: 'scan'` now match on **ticket type name**, so configure: `partout` → `fixed-product`
  partout wristband; `fredag` → `day-of-timestamp` (or fixed Friday colour); `lørdag eller søndag` →
  `day-of-timestamp`; family package items arrive as ordinary adult/child scans → `day-of-timestamp`.
- Retire the family **multiplier** hack ("vej B" in T-21-01): one barcode per person makes it double
  counting. Keep `multiplier` support in the contract for Zettle gate sales, where a family product is still
  one line.
- `cashRegisterName` equals the Billetto terminal name. Add every real terminal name to the entrance's
  `Bar.cashRegisterNames` before the festival; unmapped names must stay visible as `unmapped`, not vanish.
- Day boundary stays 04:00 Europe/Copenhagen; Billetto timestamps are UTC.

## Pre-day ordering signal (EPIC-96)

Pre-sold volume per ticket type (ticket types + sold counts from tw-tickets' `TicketPrices`, or from the
Billetto BI report via path B) feeds "order N green wristbands for Saturday morning". Treat it as a forecast
input with its own warning when stale (> 1 h) and never mix it into `soldUnits`, which is realised
consumption.

## Public live counter for the website

A `publicGet`-style route (rate-limited, no auth, `kind: 'public'` in `PUBLIC_ROUTE_ALLOWLIST` and
`auth: 'public'` in the registry) returning `{ ticketType, remaining, soldOut, updatedAt }` from a server
cache refreshed at most every 60 s. Round aggressively ("under 100 tilbage") so nobody reverse-engineers
sales; return `null` rather than stale numbers older than 10 minutes.

## Deposit refund list

Deposit = a Billetto ticket type. After the festival: list attendees of that type with `expand=data.order`,
join to the refund decision (who returned the item/site clean), and export `order id, attendee id, amount` as
CSV for Billetto support. Refunds are done by Billetto; Ops Hub only produces the list and audits who approved
each line.

## Validation checklist

- [ ] No Billetto field name, API version or event id outside the adapter/config.
- [ ] `scanSalesAvailable` and warnings distinguish "unreachable", "unconfigured" and "empty".
- [ ] Scan rules match on ticket type names; family multiplier removed for `scan` rules.
- [ ] Terminal names mapped to entrances; unmapped remain visible.
- [ ] Any new route is in `api-docs/registry.ts` and `routeGuard.ts` allowlist with the right `kind`;
      webhook routes verify `Billetto-Signature` with constant-time compare and are idempotent.
- [ ] Secrets only via Key Vault references (`BILLETTO-API-KEY`, `BILLETTO-WEBHOOK-SECRET`); never in
      `appsettings.nonsecret.json`.
- [ ] Ops Hub still persists no sales rows; caches are per-instance and short.
- [ ] Tests: fixtures with real-shaped attendee JSON (entrance + exit scans, cancelled attendee, package
      items), fail-loud paths, refill gate on `scan-sales-unavailable`.
- [ ] EPIC-21 decision file updated; tw-tickets `AI.md` note kept accurate.
