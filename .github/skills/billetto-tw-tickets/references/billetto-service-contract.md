# BillettoService contract for TwTicketsProxy

The service lives next to `Place2BookService.ts` and exposes the same three capabilities so the timers and
routes do not learn vendor details. Pseudocode below is TypeScript in the repo's ES5-lib style.

## Configuration

| Env / bicep param | Purpose |
|---|---|
| `TICKET_VENDOR` = `place2book` \| `billetto` | which service the timers instantiate (rollback switch) |
| `BILLETTO_API_KEY` (`billettoApiKey`) | `Api-Keypair` header; value from `tw-opshub-kv` secret `BILLETTO-API-KEY` |
| `BILLETTO_API_BASE` | default `https://billetto.dk/api/v3` |
| `BILLETTO_API_VERSION` | default `2026-02-10`; sent as `Billetto-Version` |
| `BILLETTO_WEBHOOK_SECRET` | only if the webhook route is deployed |
| `BILLETTO_EVENT_IDS` (optional) | comma list to restrict sync to the festival's events during the dual-vendor period |

Keep `PLACE2BOOK_*` until the 2026 reports are frozen; `Place2BookService` throws when its env is missing, so
guard its construction behind the vendor switch.

## Shapes the rest of the function expects

```ts
export interface IVendorEvent { id: string; name: string; beginAt: string; endAt: string; active: boolean; }
export interface IVendorArrival {
  arrivalId: string;        // scan log id
  scannedAt: string;        // ISO
  scannerName: string;      // Billetto terminal_name  → TicketsArrivals.kasseNavn
  ticketId: string;         // attendee id             → TicketsArrivals.ticketId
  ticketTypeName: string;   // ticket_type.name        → TicketsArrivals.ticketTypeId (+ new column ticketTypeName)
  attendeeState: string;    // sold | door_sale | ... (stored additively as attendeeState)
}
export interface IVendorPriceTier { id: string; name: string; value: number; soldTickets: number; doorSales: number; capacity?: number; }
```

## Endpoint mapping

### Events

`GET /organiser/events?limit=100` → map `name → name`, `starts_at → beginAt`, `ends_at → endAt`,
`state === 'published' → active` (`draft`, `completed`, `canceled` are not active). Note the live field
names (`name`, `starts_at`, `public`) differ from some reference pages (`title`, `startdate`, `private`).
Non-public published events may be included during setup behind a flag, but never in `TicketPrices`
feeding reports. Use `BILLETTO_EVENT_IDS` to pin the festival's events rather than syncing all 15+.

### Arrivals (the important one)

```
GET /organiser/events/{eventId}/attendees?limit=100&updated_after=<sentinel syncedAt>&expand=data.ticket_type
loop while has_more using after=<last id>
for each attendee:
  if (ADMISSION_STATES.indexOf(attendee.state) === -1) continue;   // sold, door_sale, manually_generated, mass_generated
  for each scan in attendee.scannings.data:                         // scannings is a list OBJECT, not an array
    if (scan.mode !== 'entrance') continue;                         // exits are not arrivals
    upsert TicketsArrivals { partitionKey: eventId, rowKey: scan.id,
      eventName, scannedAt: scan.scanned_at, kasseNavn: scan.terminal_name,
      numberOfItems: 1, ticketTypeId: classify(ticket_type.name), ticketTypeName: ticket_type.name,
      ticketId: attendee.id, attendeeState: attendee.state }
upsert sentinel { rowKey: '__synced', syncedAt: now }
```

Notes:

- `updated_after` is the only incremental filter; a new scan bumps `updated_at`, so this catches scans.
  Keep the five-minute TTL and the sentinel exactly as today.
- Because a cancelled attendee's old scans must stop counting, run a slower full pass (e.g. nightly, or
  every sync for events within 36 hours of start) that deletes arrivals whose attendee is no longer in an
  admission state. Upsert alone cannot un-count.
- `classify(name)`: `partout` → `partout`; names containing `familie`, `barn`, `voksen`, `fredag`,
  `lørdag`, `søndag` → `day`; others → `other`. Store the raw name too; the heuristic is for the legacy
  `TicketCategory` only.
- Expect ~10 000 attendees per festival; one sync of all events is a few hundred requests at most. Watch
  `X-Ratelimit-Cost` on the first run and set the TTL accordingly.

### Price tiers / sold counters (`TicketPrices`)

Ticket types: `GET /organiser/ticket_types?event={id}&limit=100` → `id, name, price (øre → kr), quantity,
type` (`FreeTicketType`, `FoodAndBeverageTicketType`, `BundleTicketType`; package items are separate
ticket types with price 0 inside the bundle). The nested path `/organiser/events/{id}/ticket_types` is 404.
Sold counts: the BI report (`GET /organiser/events/{id}/bi_report/`) returned 404 for TimeWinder's events on
2026-10-25, so count attendees by `ticket_type` and `state in ADMISSION_STATES` while paging arrivals (one
pass, two outputs). `doorSales` = attendees with `state = door_sale`. Event-level
`availability.available` can be stored as a sanity check.

## Webhook route (optional, after timers work)

- Route `POST /tickets/billetto-webhook`, whitelisted as `'POST /billetto-webhook'`, **not** behind
  `validateToken`.
- Verify `Billetto-Signature`: split `t=` and `v1=`, compute HMAC-SHA256 over `t + '.' + rawBody` with
  `BILLETTO_WEBHOOK_SECRET`, constant-time compare, reject if `|now - t| > 300 s`.
- Subscribe to `attendee.scanned`, `attendee.cancelled`, `order.refunded`, `order.completed`.
- On `attendee.scanned`: fetch the attendee by id (payload schema is undocumented; do not trust payload
  fields beyond the id until verified) and upsert its entrance scans. On cancel/refund: delete that
  attendee's arrivals. Respond 202 quickly; do the work before returning or queue it.
- Idempotent on scan id; retries (up to 15) are normal.

## Diagnostics

Extend `/diag` with: vendor, API version, per-event sentinel age, last `X-Ratelimit-Remaining`, last 429 time,
webhook last-received time. The gates will ask "is scanning data live?" and this is the answer.

## Dual-vendor period

2026 data (Place2Book) and 2027 data (Billetto) live in the same tables, partitioned by event id. Reports that
join on `ticketTypeId` keep working; reports that assume `numberOfItems > 1` for families must be checked.
`ImportSource` gains `'billetto'` for the upload path if historical Billetto exports are ever imported.
