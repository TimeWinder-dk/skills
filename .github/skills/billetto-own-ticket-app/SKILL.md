---
name: billetto-own-ticket-app
version: 1.0.0
releaseDate: 2026-10-25
description: |
  Decision guide and architecture for building TimeWinder's own ticket/gate app on top of Billetto instead of
  (or beside) Billetto's Access Control app: when it is worth it, the hard fact that Billetto cannot receive
  external check-ins (so an own app owns entry truth), the scan-ledger design, barcode capture on Zebra TC57
  via DataWedge or camera, offline-first validation, terminal identity for wristband accounting, exit counting,
  door-sale handoff, privacy, and reconciliation with Billetto. Use this skill whenever a task mentions egen
  billetapp, scanner-app, skannings-app, gate app, adgangskontrol, Billetto Access Control, Zebra/TC57,
  DataWedge, offline scanning, re-entry, or putting ticket scanning into the Operations Hub mobile app. Read
  billetto-platform first; pair with mobile-pwa-runtime, security-hardening and integration-resilience.
owner: TimeWinder IT
lastReviewed: 2026-10-25
---

# Own ticket app on Billetto

Billetto ships a capable scanner (Billetto app on iOS/Android/web, Billetto Access Control on Android, Multi
Scanner, terminal restrictions, exit codes, organiser keys for staff). An own app is justified only when the
gates need something Billetto's app will not do, and it comes with a price nobody can negotiate away:
**Billetto does not accept check-ins from outside.** Whatever your app scans is invisible in Billetto's
reports and in Billetto's own scanner. Decide with that in mind.

## DO NOT USE FOR

- Selling tickets, widgets or the website (`billetto-web-sales`).
- Syncing scans into tw-tickets or Operations Hub (`billetto-tw-tickets`, `billetto-ops-hub`) — this
  skill may *produce* scans; those skills consume them.
- Volunteer check-in (Volunio attendance), which is not ticketing.

## Required workflow

1. Read `billetto-platform` (limits) and its `references/api-reference.md` (attendee object, states,
   `scannings`, rate limits).
2. Run the decision table below with the gate team. If Billetto Access Control covers the need, stop here and
   write that down.
3. If building: read [references/scan-ledger-design.md](references/scan-ledger-design.md), then the Ops Hub
   skills `mobile-pwa-runtime` (Capacitor 8, Android, MSAL, offline, push) and `security-hardening`.
4. Settle the **truth model** before any code: either Billetto's app is primary and the own app is a
   read-only assistant, or the own app is the sole gate truth and Billetto's app is not used at those gates.
   Mixed use at one gate produces two half-truths.
5. Design offline-first; the field has no reliable network at the gates on Saturday morning.

## Decision table

| Need | Billetto Access Control | Own app |
|---|---|---|
| Scan barcode, reject invalid/used, per-terminal ticket-type restriction | yes | yes, with own ledger |
| Prompt staff for the wristband colour / count per scan, tied to Ops Hub products | no | yes |
| Show the scan in Billetto reports, guest list, Billetto "scanned" status | yes | **never** |
| Cross-event scanning months before the event | 36-hour window only | yes |
| Offline scanning with later sync | claimed, conflicting docs; test it | yes, by design |
| Zebra TC57 hardware scanner (DataWedge) | works as keyboard input into the app | works the same |
| Door sale at the gate | Billetto POS, one event per sale | hand off to Zettle flow in Ops Hub |
| Exit counting / people on site | universal exit code | own exit mode |
| Integration with Ops Hub wristband accounting | via API/webhook polling (minutes) | immediate |
| Cost | included | build + maintain + support on festival morning |

Typical honest outcome: Billetto Access Control at the entrances, own app only if the wristband-colour prompt
or Ops Hub coupling proves necessary after one festival on Billetto. Build the ledger first, the UI second.

## Architecture if you build it

```
Billetto (master: attendees, barcodes, states)
   │  nightly full sync + every 5 min delta (updated_after) + webhook attendee.cancelled/refunded
   ▼
Ops Hub backend: ticket projection (read-only copy: attendeeId, barcode hash, ticketTypeName, eventId, state, validFrom/To)
   │  signed device sync (delta by updated_at), scan upload (idempotent by scanId)
   ▼
Gate app (Ops Hub Capacitor app, Android/TC57): local SQLite/IndexedDB copy of valid barcodes for the event,
   local scan ledger (scanId uuid, barcode, terminalName, scannedAt, mode entrance/exit, result), outbox
   ▼
Ops Hub scan ledger (source of entry truth) → ProductSaleRow source 'scan' → wristband accounting / refill
```

Key rules:

- **Projection, not master.** Copy the minimum: attendee id, barcode (store a hash on the device, verify
  by hash), ticket type name, event id, state, order id. No names or e-mails on devices unless the gate flow
  truly needs them; if it does, keep them server-side and look up on demand.
- **Validation is local and deterministic**: barcode exists, state admissible, ticket type allowed on this
  terminal/day, not already entered (or re-entry policy satisfied), event window open. Return a result code the
  staff can act on in one glance; log every result, including rejections.
- **Terminal identity = Ops Hub entrance name.** The device chooses from `Bar.cashRegisterNames`, so
  wristband accounting maps without manual work. No free-text terminal names.
- **Idempotent scans**: scanId is a UUID generated on the device; the server dedups; the outbox retries with
  backoff; the UI shows "N scans not yet synced".
- **Conflicts**: two devices offline can both admit the same barcode. Resolve server-side as "first scannedAt
  wins", flag the second, and show the gate lead a daily conflict list. This is the cost of offline.
- **Door sales** stay in Zettle; the app offers a "sold at gate" shortcut that opens the Zettle flow and does
  not create a fake ticket.
- **Exit mode** mirrors Billetto's exit code: count only, no validation.
- **Security**: device gets a short-lived token through the existing Ops Hub mobile auth (MSAL/session),
  never the Billetto API key. Scan upload routes are bearer-gated like every other Ops Hub route.
- **Reconciliation**: after each day, compare the ledger with Billetto's `scannings` where Billetto's app
  was also used, and with ticket-type sold counts; publish the diff in the delivery overview instead of hiding
  it.

## Hardware notes *(point in time)*

Used Zebra TC57 units were offered to TimeWinder (Billetto confirmed they work as long as Google Play or APK
install is possible). DataWedge can emit scans as keystrokes or intents; keystroke mode works with a plain
focused input and needs no camera plugin. For phones, `@capacitor-mlkit/barcode-scanning` (verify version
against Capacitor 8) is the usual camera path. Billetto can lend a few scanners; SkanCode is their reseller.

## Validation checklist

- [ ] The decision table was filled in with the gate team and the chosen truth model is written down.
- [ ] Device holds a minimal projection (hashes, ids, types, states), no API keys, no unnecessary personal data.
- [ ] Validation rules are local, deterministic and unit-tested with real-shaped attendee fixtures
      (sold, door_sale, cancelled after scan, package items, wrong day, re-entry).
- [ ] Scan ids are UUIDs; upload is idempotent; outbox survives app restart; sync status visible to staff.
- [ ] Terminal identity is picked from Ops Hub entrance names.
- [ ] Offline conflict policy implemented and surfaced to the gate lead.
- [ ] Reconciliation report against Billetto exists before the first festival day.
- [ ] Rehearsed on the actual TC57 devices with the festival-morning network, not only on Wi-Fi at home.
