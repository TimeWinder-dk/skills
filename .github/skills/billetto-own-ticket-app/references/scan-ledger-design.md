# Scan ledger design — own gate app on Billetto

## Entities

### Ticket projection (server, read-only copy of Billetto)

| Field | Source | Notes |
|---|---|---|
| `attendeeId` | attendee `id` | primary key |
| `eventId` | attendee `event` | |
| `barcodeHash` | SHA-256 of `barcode` | the device never needs the clear barcode; compare hashes |
| `ticketTypeName` | `ticket_type.name` | drives terminal restrictions and wristband rules |
| `state` | attendee `state` | admissible: `sold, door_sale, manually_generated, mass_generated` |
| `orderId` | `order` | for support lookups |
| `updatedAt` | `updated_at` | delta sync cursor |
| `provenance` | `billetto`, `syncGeneration` | follow the Ops Hub provenance model (ownership_state, last_imported_at) |

Sync: nightly full pass per event plus a 5-minute delta with `updated_after`. Webhooks
`attendee.cancelled`, `attendee.updated`, `order.refunded` trigger an immediate delta for that attendee.

### Scan ledger (server, TimeWinder-owned truth)

| Field | Notes |
|---|---|
| `scanId` | UUID generated on the device |
| `attendeeId` / `barcodeHash` | what was scanned (hash kept even when unknown, for forensics) |
| `terminalName` | one of the entrance names from Ops Hub |
| `deviceId` | registered device |
| `scannedAt` | device clock, ISO; server also stores `receivedAt` |
| `mode` | `entrance` / `exit` |
| `result` | `admitted`, `rejected:unknown`, `rejected:state`, `rejected:type-not-allowed`,
  `rejected:already-entered`, `rejected:window`, `admitted:re-entry`, `conflict` |
| `wristbandProductId` | optional, if staff confirms the colour |
| `operator` | staff identity from the app session |

Append-only. Corrections are new rows with `supersedesScanId`.

### Device sync state

| Field | Notes |
|---|---|
| `deviceId`, `eventIds[]`, `terminalName` | registration |
| `projectionCursor` | last `updatedAt` received |
| `outboxCount`, `lastUploadAt` | shown to staff |

## Local validation (device)

```
lookup = projection.byBarcodeHash(hash(code))
if (!lookup)                              → rejected:unknown
if (!ADMISSIBLE.has(lookup.state))        → rejected:state
if (!terminal.allowedTypes.matches(lookup.ticketTypeName, today)) → rejected:type-not-allowed
prior = ledger.entriesFor(lookup.attendeeId)
if (prior.length && !policy.reentry)      → rejected:already-entered
if (prior.length && policy.reentry)       → admitted:re-entry (no wristband)
if (!event.window.contains(now))          → rejected:window
→ admitted (prompt wristband colour if rule says so)
```

`terminal.allowedTypes` is the own-app equivalent of Billetto's terminal restrictions; configure it per day in
Ops Hub, not on the device.

## Offline and conflicts

- Projection and ledger live in SQLite (Capacitor) or IndexedDB (PWA). Projection size: ~10 000 rows per
  festival, trivial.
- Outbox uploads in batches of 100 with exponential backoff; duplicates are dropped server-side by `scanId`.
- Conflict detection server-side: same `attendeeId`, two `admitted` rows from different devices →
  keep the earliest as truth, mark the later `conflict`, notify the gate lead. The device that made the later
  scan gets the correction on its next sync so staff can learn the pattern.
- Clock drift: the server records `receivedAt` and flags devices whose `scannedAt` differs from server time by
  more than 2 minutes at upload.

## Interfaces to the rest of the family

- Ops Hub reads the ledger as `ProductSaleRow` with `source: 'scan'`, `cashRegisterName = terminalName`,
  `productName = ticketTypeName`. The wristband accounting then works exactly as with Billetto's scans.
- The delivery overview shows a "ledger vs Billetto" reconciliation panel: entries in the ledger not in
  Billetto `scannings` (expected when the own app is primary) and entries in Billetto not in the ledger
  (should be zero if the own app is primary at that gate).
- Door sales remain Zettle rows; the app never emits a sale.

## Privacy

- Devices hold hashes and ticket types, not people. Name lookup, when the gate needs it (lost ticket), is a
  server call under the operator's identity and is audited.
- Ledger retention: keep scans for the accounting period, then anonymise `barcodeHash` and `attendeeId` to
  the ticket type and day.

## Rehearsal plan

1. Private test event in Billetto with generated free tickets (they count towards capacity; use a test
   event).
2. Sync projection; scan on two TC57 devices offline; bring one online, then the other; verify conflict
   handling and outbox drain.
3. Cancel a ticket in Billetto after it was scanned; verify the projection update and the ledger's audit row.
4. Compare the delivery overview wristband numbers with a manual count.
