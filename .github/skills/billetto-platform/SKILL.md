---
name: billetto-platform
version: 1.0.0
releaseDate: 2026-10-25
description: |
  Shared knowledge of the Billetto ticketing platform as TimeWinder uses it from festival 2027 on: account/plan
  and fee model, event → ticket type → package/group model, Basket, campaigns, target groups and codes, scanning
  (Billetto app, Access Control, Multi Scanner, terminal restrictions, exit codes), POS/Adyen, reports, the
  organiser API v3 and webhooks, and the hard platform limits (no external check-in write-back, no partial
  refunds, one PDF per order). Use this skill whenever a task mentions Billetto, billetto.dk, billetter/ticket
  sales, ticket types, familiepakker, Basket, access codes, Adyen payouts, scanning at the gates, or the
  TimeWinder ticket account — even if the user only says "billetsystemet". Always read it before the more
  specific skills billetto-web-sales, billetto-tw-tickets, billetto-ops-hub and billetto-own-ticket-app.
owner: TimeWinder IT
lastReviewed: 2026-11-10
---

# Billetto Platform

Billetto is TimeWinder's ticket vendor from the 2027 festival (Pinsen 14–16 May 2027). Place2Book served
2026 and is being retired. This skill is the single, shared description of what Billetto is and is not, so
every integration (website, tw-tickets, Operations Hub, a possible own scanner app) builds on the same facts
and the same limits.

Facts here were verified on 2026-10-25 against api.billetto.com, support.billetto.com and the mail dialogue
with Billetto in tha@timewinder.dk. Anything marked *(point in time)* can change; re-verify before you rely on
it for money or capacity decisions.

## DO NOT USE FOR

- Place2Book or Zettle behaviour (see tw-tickets `AI.md` and the Operations Hub EPIC-21 decision).
- Volunteer/crew access — that is Volunio, not a ticket. A volunteer wristband is not a Billetto ticket.
- Writing code without the specific integration skill: this skill explains the platform, the sibling skills
  explain where the code goes.

## Required workflow

1. Classify the task: **selling** (website, campaigns, prices) → `billetto-web-sales`; **data in** (scans,
   sales, arrivals into TimeWinder systems) → `billetto-tw-tickets` and/or `billetto-ops-hub`; **scanning
   hardware/app** → `billetto-own-ticket-app`; **account/plan/money questions** → stay here.
2. Read [references/api-reference.md](references/api-reference.md) before touching the API or webhooks.
3. Read [references/timewinder-account.md](references/timewinder-account.md) for the current state of
   TimeWinder's account: events, packages, codes, open decisions, contacts. Treat it as a dated snapshot.
4. Check every assumption against the **hard limits** below. Most "smart" ideas die on one of them.
5. Name the owner of each data fact (Billetto, tw-tickets, Operations Hub) before designing a flow; Billetto is
   the master of orders, attendees and barcodes, full stop.

## Platform model

```
Organiser account (TimeWinder, "IT Systemer på tværs")
└── Event (public/private, state, start/end, one per sales context)
    ├── Ticket type   price, quantity, sells_from/to, min/max per order, vip, fee_included,
    │                 admission/merchandise/addon/donation flags, hidden
    ├── Ticket group  shared capacity across several ticket types ("lørdag eller søndag")
    ├── Ticket package bundle of items; each item gets its own ticket + barcode by default;
    │                 composition is frozen at creation (only the package count can change)
    ├── Campaigns     early bird, Easy Discounts per audience, access codes, unique codes
    ├── Scanner settings  terminal restrictions (which ticket types a named terminal may admit)
    └── Attendees     one per ticket: barcode, state, ticket_type, order, scannings[]
Account-level: Basket (up to 5 events in one order), target groups/audiences, Smart Links & QR,
               widgets, chat widget, webhooks, API keys, Adyen payout, POS
```

What that means in practice:

- **Separate events are fine.** Entré, camping and nostalgicamping stay separate events; Basket joins them
  into one online order. Basket only works on *public* events and never inside a widget.
- **A family ticket is a package.** "2 voksne + 2 børn" yields four tickets and four barcodes. That removes
  the old one-scan-per-family problem. Make one package per child count (2+1, 2+2, 2+3); you cannot edit the
  item list afterwards.
- **Day logic is a scanner concern.** "Fredag" vs "lørdag/søndag" admission is enforced by terminal
  restrictions that staff pick manually on the device; nothing flips automatically at midnight.
- **Free tickets must be free.** New events default to a fee on 0 kr tickets until the account rule is
  applied; check it per event.
- **Test purchases are real purchases.** Stop before payment when testing on public events, or use a
  private test event and "Generer billetter" (free, counts towards capacity).

## Money *(point in time)*

| Item | Value | Source |
|---|---|---|
| Standard fee | 3,9 % + 6,99 kr + VAT per ticket, buyer pays by default | billetto.com pricing, Q&A sheet |
| Offered to TimeWinder (Jan 2025) | 2,5 % + 4,50 kr + VAT; 0 kr free tickets; 0 kr admin | Q&A sheet from Billetto |
| Final negotiated rate | Set on the account 2026-02-17; exact figure not in mail — ask Carsten/Mikkel | mail thread |
| MobilePay | 2,99 kr + VAT per transaction | Q&A sheet |
| Payout | 3–5 working days after the event; Adyen enables rolling weekly/monthly payouts | Billetto |
| POS | Adyen terminal ≈ €199 one-off; Stripe quote Dec 2025 was 1,4 % + 0,0175 kr/transaction | Billetto |
| Plan | Growth (1 000+ tickets/year): custom pricing, prepayment, CSM | billetto.com |

Volumes to size against: normal year ≈ 9 500 tickets, ≈ 2 200 online; 2025 ≈ 5 200 / 2 280; ≈ 60 tickets
per year above 1 000 kr (TimeWinder asked for a lower percentage on those).

## Hard limits (design around these, do not fight them)

1. **No external check-in.** The API and webhooks can *read* scans; nothing can register a scan *into*
   Billetto from outside. An own scanner app therefore owns its own entry truth (see
   `billetto-own-ticket-app`).
2. **Refunds are whole tickets, done in Billetto, and self-service closes when the event ends.** A deposit
   is modelled as a separate ticket type and refunded by Billetto support from a list TimeWinder sends after
   the festival. Adyen may already have paid the money out, so agree a hold-back first.
3. **One PDF per order.** No setting sends one file per ticket; organisers can download single tickets, and
   buyers can transfer single tickets through Billetto Wallet (barcode re-issued). TimeWinder keeps its own
   PDF splitter for now. For tickets TimeWinder issues itself (exhibitor free tickets) an **own ticket design
   with the barcode as QR is allowed** (Billetto, 2026-11-05) — see limit 10.
4. **Basket is online-only and public-only.** Door sales (billetkontor/POS) sell one event at a time.
5. **Multi Scanner shows only events starting within 36 hours**, so cross-event scanning cannot be
   rehearsed months ahead; use two test events inside the window.
6. **Ticket types with any order cannot be deleted**, only hidden.
7. **Rate limiting is cost-based**, not request-based; a 429 means pause at least a minute and honour
   `X-Ratelimit-Retry-After`.
8. **The public event endpoint has no per-type availability** (only a boolean and a minimum price), still
   requires the key, and 404s for events with `public: false`. Live "tickets left" counters need the
   organiser API behind your own backend; the only count it exposes is the event-level
   `availability.available`.
9. **"Published" is not "public".** All 2027 events were published but `public: false` on 2026-10-25.
   Basket, the public endpoint and marketplace exposure need `public: true`; flip it deliberately before
   launch, not during.
10. **The API cannot create orders or tickets.** Its only writes are target-group members (unique codes),
    target-group imports and webhooks (key-pair webhook creation fixed by Billetto 2026-11-03). "Generer
    billetter" is dashboard-only, 20 per batch. The dashboard's internal
    `POST /api/organizer/events/{id}/tickets/generate` needs a dashboard session or Billetto-issued OAuth token
    — 401 with the key pair, and Billetto confirmed in writing (2026-11-05) that no setting or key type opens
    it; a public equivalent would be a new, unscheduled product feature. Two supported ways to hand out
    tickets automatically:
    - **Own barcodes + CSV import** (enabled on the account 2026-11-05): TimeWinder generates the barcode and
      its own PDF/QR, then imports the barcodes into Billetto, where they scan normally (verified end to end
      2026-11-04 incl. lowercase-prefixed barcodes such as `vtb11223344557`). The import is manual in the
      dashboard; the ticket is not valid at the gate until imported. Used for exhibitor free tickets —
      see `power-automate-specieltickets` (repo `TimeWinder-dk/power-platform`) and `docs/billetto/README.md`.
    - **A unique code** the person redeems themselves (target group + campaign), e.g. paid market stalls.

## Where things live in the organiser UI *(point in time)*

| Need | Path in My Billetto |
|---|---|
| API keys, webhooks | Integrate → Developers |
| Free/preview ticket to yourself | Administrer event → Deltagere → Billethåndtering → Generer billetter |
| Ticket PDF banner (480 × 730 px PNG/JPG) and order-mail text | Design & tilpasninger → Billetter og ordrer |
| Smart Links & QR, access codes, Easy Discounts, campaigns | Rabatter & kampagner |
| Single-event / account widget code | Marketing → Widget (event) and Sell → Widget (account) |
| Terminal restrictions | Event → Skannerindstillinger → Begrænsninger på terminal — a new ticket type must be added to every gate terminal, otherwise "Terminalen X er ikke autoriseret til at scanne billettypen …" |
| CSV import of own tickets | Event → Deltagere → Billethåndtering → Upload billetter (`…/organizer/tickets/import`): one product type per file; map **every** column in its dropdown (no auto-mapping; the dropdown shows blank after selection — Billetto bug); the red text is the first *unmapped* field (external id → e-mail → barcode → name), not a data error |
| Hide a ticket type behind a code | Ticket type → Advanced settings → hidden; only then does the campaign effect "Lås billettyper op" appear (also with a target-group condition) |
| Organiser key for scanning staff | Profile → Event Manager |
| Private test events | Event information → Privat (publish as normal; only you and Billetto see it) |

## Validation checklist

- [ ] The task is routed to the right sibling skill (web sales, tw-tickets, Ops Hub, own app).
- [ ] No design assumes external check-in, partial refunds, per-ticket PDFs, Basket in a widget or Basket at
      the door.
- [ ] Capacity and money figures are re-verified, not copied from this snapshot.
- [ ] Family/group tickets are modelled as packages (one barcode per person), never as a multiplier.
- [ ] Test flows use private events or stop before payment; generated free tickets are accounted for in
      capacity.
- [ ] Billetto is named as master for orders, attendees and barcodes; TimeWinder systems hold projections.
