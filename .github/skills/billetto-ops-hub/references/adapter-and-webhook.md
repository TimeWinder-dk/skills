# Operations Hub ↔ Billetto — adapter, webhook and wiring details

## Existing building blocks (Ops Hub repo, verified 2026-10-25)

| Piece | Where | Role |
|---|---|---|
| `ProductSaleRow` | `shared/src/dto.ts` | vendor-neutral sale/scan row (`source: 'zettle' \| 'online' \| 'scan'`) |
| Wristband rules | `shared/src/wristbandAccounting.ts` | pure config + rule lookup, `SETTING_WRISTBAND_ACCOUNTING` app setting |
| Scan adapter | `backend/src/services/ticketArrivalsStore.ts` | reads tw-tickets `TicketsArrivals` via `@azure/data-tables` + managed identity; 60 s cache |
| Sales client | `backend/src/services/ticketSalesClient.ts` | tw-tickets `GET /sales` (Zettle + aggregated online) |
| Config | `backend/src/config.ts` → `ticketArrivals`, `ticketSales` | env `TICKET_ARRIVALS_ACCOUNT_NAME/TABLE`, `TW_TICKETS_*` |
| Consumers | `deliveryOverviewService.ts`, `salesMapping.ts`, `refillEngine.ts` | must stay vendor-agnostic |
| Webhook precedents | `functions/gatewayApiWebhook.ts`, `acsSmsWebhook.ts`, `teamsBot.ts` | anonymous route + provider validation; listed in `routeGuard.ts` `PUBLIC_ROUTE_ALLOWLIST` with `kind: 'webhook'` |
| Route contract | `api-docs/registry.ts`, `api-docs/schemas.ts`, `docs/api/openapi.json` | every route registered; `auth: 'webhook'`/`'public'` for non-bearer |
| Secrets | `tw-opshub-kv` (prod), `tw-opshub-udd-kv` (training) | `PLACE2BOOK-API-TOKEN` pattern: secret in vault, consumer reads env |

## Path B: `BillettoArrivalsStore`

Same public surface as `TicketArrivalsStore` so `deliveryOverviewService` cannot tell them apart:

```ts
interface ScanSource { fetchScans(params: { from: string; to: string }): Promise<ProductSaleRow[]>; }
```

Implementation sketch:

- Config block `billetto?: { apiBase; apiKey; apiVersion; eventIds: string[]; webhookSecret? }` populated
  from `BILLETTO_API_BASE`, `BILLETTO_API_KEY` (Key Vault ref), `BILLETTO_API_VERSION` (default
  `2026-02-10`), `BILLETTO_EVENT_IDS`.
- For each configured event: page `GET /organiser/events/{id}/attendees?limit=100&expand=data.ticket_type`
  with `updated_after` = `from` minus one day (scans can be late), follow `after` cursors, collect
  `scannings.data[]` (a list object, verified live) where `mode === 'entrance'` and `scanned_at` within
  `[from, to]`, attendee `state` in `sold | door_sale | manually_generated | mass_generated`. Ticket types
  come from `GET /organiser/ticket_types?event={id}`; the header is `Api-Keypair: <id>:<secret>`.
- Map to `ProductSaleRow` (see SKILL.md). `productName` = `ticket_type.name` trimmed.
- Honour rate-limit headers: on 429 throw `ServiceUnavailable` so the overview reports "unavailable"; do not
  retry inside the request.
- Cache per `from|to` for 60 s like today; the refill engine asks repeatedly.
- Throw on unreachable; return `[]` only for "reachable, no scans" — the caller distinguishes the two.

Choose B only with a written reason in the EPIC-21 decision file: it duplicates the vendor client that
tw-tickets owns.

## Path C: webhook route

- File `backend/src/functions/billettoWebhook.ts`, route `billetto/webhook`, `authLevel: 'anonymous'`.
- Add to `PUBLIC_ROUTE_ALLOWLIST`: `'billetto/webhook': { kind: 'webhook', reason: 'Billetto event
  notifications — HMAC-validated (Billetto-Signature).' }` and to `registry.ts` with `auth: 'webhook'`;
  `test/routeGuard.test.ts` and the OpenAPI contract test enforce both.
- Verification: parse `Billetto-Signature` (`t=`, `v1=`), HMAC-SHA256 over `${t}.${rawBody}` with the
  secret from `BILLETTO_WEBHOOK_SECRET`, `crypto.timingSafeEqual`, reject if the timestamp is older than
  5 minutes. Read the raw body before any JSON parsing.
- Behaviour: respond 202 immediately after verification; record `{ type, receivedAt }` in a small durable
  marker (app setting/table) that invalidates the scan cache or bumps a "data changed" timestamp shown in the
  delivery overview. Do **not** store attendee rows: that would make Ops Hub a sales store.
- Retries: Billetto retries 15 times; the handler must be idempotent (same event id → same effect).
- Suppression: honour `outboundSuppressed()`/review mode only for anything the handler sends out; inbound
  verification itself always runs.

## Config and infra

| Setting | Secret? | Where |
|---|---|---|
| `BILLETTO_API_KEY` | yes | Key Vault `BILLETTO-API-KEY`, referenced from `main.bicep`/`main.bicepparam` |
| `BILLETTO_WEBHOOK_SECRET` | yes | Key Vault `BILLETTO-WEBHOOK-SECRET` |
| `BILLETTO_API_BASE`, `BILLETTO_API_VERSION`, `BILLETTO_EVENT_IDS` | no | `infra/appsettings.nonsecret.json` + bicep |

The training environment (`udd`) gets no Billetto key by default; the code must report "unconfigured" there.

## Tests to write

- Fixture `billetto-attendees.page1.json` with: a sold attendee with one entrance scan, one with entrance +
  exit scans, a cancelled attendee with an old scan, three package items from one order, a `door_sale`.
- Expect: 4 scan rows (exit ignored, cancelled ignored, package items each counted), `cashRegisterName` =
  terminal, `productName` = ticket type name.
- Rate-limit fixture: 429 with `X-Ratelimit-Retry-After` → `ServiceUnavailable` → `scanSalesAvailable:false`.
- Webhook: valid signature → 202; bad signature → 401; stale timestamp → 401; duplicate delivery → no second
  effect.
- Existing `backend/test/wristbandAccounting.test.ts` must stay green; add a case with ticket-type matching
  (`partout` fixed product, `lørdag eller søndag` by timestamp) and no multiplier on scans.
