---
name: economic-api
version: 1.0.0
releaseDate: 2026-11-09
description: |
  Working knowledge of Visma e-conomic's REST API (restapi.e-conomic.com) as TimeWinder uses it: the two-token
  auth model (app secret + agreement grant), how an agreement grants access to the app "Timewinderudtræk", where
  the tokens live (Key Vault tw-opshub-kv), the resource model (customers, products, draft → booked invoices,
  payment terms, VAT zones, layouts), filtering/pagination/idempotency rules, roles, error shapes and the hard
  limits (no versioning, booked invoices are immutable, demo tokens are GET-only, no published rate limit). Use it
  whenever a task mentions e-conomic, economic, bogføring/regnskab via API, fakturakladde, fakturering af
  personaleforeninger, kunder/varer i e-conomic, or the headers X-AppSecretToken / X-AgreementGrantToken.
owner: TimeWinder IT
lastReviewed: 2026-11-09
---

# e-conomic API

Visma e-conomic is TimeWinder's accounting system; the bookkeeper works in it and the shared mailbox
`bogholderi@timewinder.dk` is the human entry point. This skill is the single description of how TimeWinder
systems (Operations Hub first) talk to e-conomic, so every integration shares the same facts, the same token
handling and the same limits. First planned use: **invoice drafts for personaleforeningsbilletter**
(Operations Hub `docs/ai/decisions/EPIC-108-PERSONALEFORENINGSBILLETTER.md`, D9).

Facts were verified 2026-11-09 against `restdocs.e-conomic.com`, `restapi.e-conomic.com/schema/*.json`, the demo
agreement (`?demo=true`) and `e-conomic.com/developer/*`. Anything marked *(point in time)* can change; re-verify
before relying on it for money.

## DO NOT USE FOR

- Deciding **what** to invoice or at which price — that is the owning feature's design (e.g. EPIC-108 D8).
- Billetto money (fees, payouts, Adyen) → `billetto-platform`.
- The legacy SOAP API (`api.e-conomic.com/secure/api1/...`) — never start new work on it.
- Booking, crediting or deleting booked documents on behalf of the bookkeeper without an explicit owner
  decision: Ops Hub creates **drafts**; a human books.

## Treat the integration as a spike first (owner, 2026-11-09)

Nothing production-shaped is built against e-conomic until a time-boxed spike has answered the questions in
[references/timewinder-agreement.md § Spike](references/timewinder-agreement.md#spike-plan-before-any-production-code)
on TimeWinder's real agreement (or a 14-day trial): token/grant flow, app role, master data, create → book →
pay → read `remainder`, filter on `references.other`, failure modes, and how long the bookkeeper's bank
reconciliation takes to show a payment. The spike ends with a short written result in Operations Hub
`docs/ai/log/` and an updated agreement snapshot; only then are build tasks estimated.

## Required workflow

1. Read [references/rest-api-reference.md](references/rest-api-reference.md) before writing a single request:
   headers, pagination, filter syntax, error shape, schemas.
2. Read [references/timewinder-agreement.md](references/timewinder-agreement.md) for the state of TimeWinder's
   app and agreement: token names in Key Vault, what is still missing, master data to map. Dated snapshot.
3. Decide the write boundary: **draft invoice** (default, reversible, bookkeeper books) vs. booked invoice (needs
   owner + bookkeeper go). Never book from code in v1.
4. Every write carries an `Idempotency-Key` and is logged with the e-conomic `logId` on failure.
5. Put the outbound call behind one client module with the repo's resilience choke point (Operations Hub:
   `resilientFetch` + `outboundSuppressed()`; see "Wiring in Operations Hub").
6. Run the validation checklist at the end.

## Platform model

```
Developer agreement (TimeWinder, developer.e-conomic.com)
└── App "Timewinderudtræk"   AppPublicToken (not secret), AppSecretToken (secret), role (SuperUser/Sales/…)
        │  InstallationURL  …/requestaccess.aspx?appPublicToken=<public>   ← an admin of the ACCOUNTING agreement opens it
        ▼
Accounting agreement (TimeWinder's books)  ──grants──►  AgreementGrantToken (per app × agreement, long-lived)
└── Customers (customerNumber, customerGroup, vatZone, paymentTerms, currency, email, CVR/EAN, contacts)
    Products  (productNumber ≤ 25 chars, productGroup → sales account, salesPrice, unit)
    Invoices  drafts (editable, deletable) ──book──► booked (immutable; credit note to undo)
              sent / paid / unpaid / overdue / notDue / totals · pdf per draft or booked
    Master data: payment-terms, vat-zones, layouts, units, customer-groups, product-groups, employees
    Journals/vouchers, orders, quotes, suppliers, accounts, accounting-years (not needed for tickets)
```

Every request needs **both** tokens. The app secret says *who the software is*; the grant token says *whose
books*. Lose the grant token and you re-run the installation URL — nothing else helps.

## Hard facts and limits (design around these)

1. **Two headers, always:** `X-AppSecretToken`, `X-AgreementGrantToken`, plus `Content-Type: application/json`.
   URL-parameter auth is not supported. Wrong or revoked token ⇒ `401`; insufficient app role ⇒ `403`.
2. **The grant token is obtained by a human**, once, by opening the InstallationURL while logged in as an
   administrator of the accounting agreement (click *Administrer* on the agreement first). It is shown on screen
   or sent to the app's RedirectURL as query parameter `token`. There is no API to fetch it.
3. **No API versioning.** The root always serves the latest; endpoints are labelled stable/experimental/
   deprecated on `GET /`. Pin nothing, but keep the client tolerant of added fields.
4. **Drafts are the safe write.** `POST /invoices/drafts` creates; `PUT`/`DELETE` change or remove; `POST
   /invoices/booked` with `{ draftInvoice: { draftInvoiceNumber } }` books (optionally `sendBy: "Email" | "EAN"`).
   Booked invoices cannot be edited or deleted — only credited.
5. **Required on a draft:** `date` (YYYY-MM-DD), `currency`, `customer.customerNumber`, `layout.layoutNumber`,
   `paymentTerms.paymentTermsNumber`, `recipient.name`, `recipient.vatZone.vatZoneNumber`. Lines are optional
   at schema level but an invoice without lines is useless; a line with a `product` must also have `quantity`
   and `description`. Totals (`netAmount`, `vatAmount`, `grossAmount`) are **read-only** — e-conomic computes
   them from `quantity × unitNetPrice` and the product group's VAT.
6. **Required on a customer:** `name`, `currency`, `customerGroup`, `vatZone`, `paymentTerms`. `email`, `ean`,
   `corporateIdentificationNumber` (CVR), `publicEntryNumber`, address fields are optional.
7. **Null by omission.** To clear a property leave it out of the body; sending `null` usually fails validation.
8. **Idempotency:** header `Idempotency-Key` (your unique value) on writes; a retry with the same key returns
   the cached response for one hour with `X-ResultFromCache: true`. Not allowed on GET.
9. **Demo tokens** (`demo`/`demo`, or `?demo=true`) only allow **GET** — you cannot rehearse a POST without a
   real (or 14-day trial) agreement.
10. **Pagination:** `pagesize` (default 20, max 1000) + `skippages` (zero-based); the `pagination` object has
    `results`, `nextPage`, `lastPage`. **Filtering:** `?filter=prop$op:value` with `$eq $ne $gt $gte $lt $lte
    $like $in $nin $null $and $or`, only on properties marked `filterable`; URL-encode and escape `$`, `(`, `*`
    with `$`. **Sorting:** `?sort=-name,age`.
11. **Errors are structured:** `400 { message, developerHint, logId, errors: { <field>: { errors: [{ errorCode,
    message, value }] } } }`, lines carry `arrayIndex`. Always log `logId` — e-conomic support needs it.
12. **Rate limit *(point in time)*:** the REST docs publish **no numeric limit** and no limit headers (checked
    2026-11-09). Be polite anyway: one request at a time per agreement, back off on `429`/`5xx`, never poll.
13. **Roles:** the app's role decides access. `Sales` or `SuperUser` reach customers, invoices, products,
    customer-groups, `self`; `Bookkeeping` does **not** reach customers/invoices. Check the role before blaming
    tokens for a `403`.
14. **Sending invoices by e-mail** (`sendBy: "Email"` at booking) requires the agreement's contact e-mail and a
    recipient on the customer (default invoice e-mail or a contact with *Receive invoices*). EAN/NemHandel needs
    CVR on the agreement + EAN on the customer. Both are bookkeeper decisions — the ticket flow leaves
    `sendBy` out and lets the bookkeeper send.
15. **Payment status is read, never written, by us.** A booked invoice carries `remainder` /
    `remainderInBaseCurrency` (0.00 ⇒ fully paid), `grossAmount`, `dueDate`, `customer.customerNumber`,
    `references.other`; `/invoices/paid`, `/invoices/unpaid`, `/invoices/overdue` are pre-filtered views.
    `references.other` **is filterable** on `/invoices/booked` (schema + live check 2026-11-09), so an order
    reference written into `references.other` at draft time is the join key. `remainder`, `grossAmount` and
    `dueDate` are **not** filterable — fetch the matched invoice and compare in code. A payment shows up only
    after the bookkeeper has registered/reconciled it in e-conomic; "booked" ≠ "paid", and a credit note changes
    the picture — re-read before acting.

## Wiring in Operations Hub (pattern)

Follow `backend/src/services/sponsorTickets/sponsorFlowClient.ts` as the template for the choke point:

- **Config:** app settings `ECONOMIC_APP_SECRET_TOKEN` and `ECONOMIC_AGREEMENT_GRANT_TOKEN` as **Key Vault
  references** (`infra/main.bicep`) to secrets `ECONOMIC-APP-SECRET-TOKEN` / `ECONOMIC-AGREEMENT-GRANT-TOKEN`
  in `tw-opshub-kv`; `ECONOMIC_BASE_URL` default `https://restapi.e-conomic.com`. Unset ⇒ the feature reports
  `configured: false` and does nothing (no demo fallback in production code).
- **Client module** `services/economic/economicClient.ts`: `resilientFetch` with a 15 s timeout, headers from
  config, `Idempotency-Key` on every POST/PUT, `outboundSuppressed()` checked right before the call (Store Review
  Mode + integrations kill switch), structured outbound log line `provider: 'economic'`, `operation:
  'invoice-draft.create'` etc. Never log tokens, never return the tokens or raw e-conomic bodies to a browser.
- **Mapping layer** `services/economic/economicMapping.ts`: pure functions from the domain (an order of
  tickets) to the e-conomic body, unit-tested with fixtures. Keep e-conomic field names **only** here and in the
  client.
- **Reconciliation:** store `draftInvoiceNumber` (and later `bookedInvoiceNumber`) on the originating record
  (SharePoint register `Ordre` row / audit) so a failed or duplicated draft can be found. A draft that already
  exists for the same order reference (`references.other`) is not created twice: query
  `GET /invoices/drafts?filter=references.other$eq:<ref>` first (verify `filterable` on the live schema — if it
  is not, rely on the idempotency key + stored number).
- **Audit:** `writeAudit()` with customer number, draft number, net total and the order reference; never the
  tokens or the full body.
- **Tests:** fixture-based unit tests of the mapping; client tests with the fetch seam; no live calls in CI.

## Recipe: payment gate ("generate tickets only when paid")

1. The order reference (`PF-<år>-<præfiks>-<nr>`) is written into `references.other` on the draft the moment it
   is created (by Ops Hub, or typed by the bookkeeper into *Anden reference* if the invoice is made by hand).
2. Reconciliation job (timer, e.g. every 30 min in office hours, plus on demand): for each order with
   `paymentStatus ≠ Betalt`: `GET /invoices/booked?filter=references.other$eq:<ref>`.
   - 0 hits ⇒ still *Anmodet / ikke bogført*. 2+ hits ⇒ *Afvigelse* (never pick one).
   - 1 hit ⇒ fetch `/invoices/booked/:n`; require `currency == DKK`, `customer.customerNumber` == the order's
     customer, `grossAmount` == the order total (tolerance 0.00), `remainder == 0`. All true ⇒ **Betalt**
     (store `bookedInvoiceNumber`, `paidVerifiedAt`); `remainder > 0` ⇒ *Delvist betalt/Afventer*.
   - Any transport/5xx/401/403 ⇒ status *Ukendt* for this run; never "assume paid", never clear a previous Betalt.
3. Credit notes: if a later run finds the invoice credited (`/invoices/booked` shows a matching negative or
   `remainder` moves), raise *Afvigelse* for a human — do not auto-revoke delivered tickets.
4. Every transition is audited with invoice number, amounts and the e-conomic `logId` on errors.

## Recipe: draft invoice for a personaleforening (EPIC-108 D9 alternative)

1. Resolve the customer: the SharePoint list *Personaleforeninger* gets a column `EconomicKundenr`
   (customerNumber). Empty ⇒ Ops Hub looks up `GET /customers?filter=corporateIdentificationNumber$eq:<CVR>`;
   still nothing ⇒ create the customer (`POST /customers` with the agreed `customerGroup`, `vatZone`,
   `paymentTerms`, `currency: "DKK"`, `email` = first address from `Mails`, `ean`, address) and write the
   number back to the list (or ask the bookkeeper — owner decision).
2. Products: one e-conomic product per ticket variant (`PF-VOK`, `PF-BRN`, `PF-FAM` — final numbers agreed with
   the bookkeeper so VAT and sales account are right). `unitNetPrice` = the discounted ticket price **excl.
   VAT if the product group is VAT-liable** — confirm with the bookkeeper whether ticket prices are VAT-exempt
   (cultural event) before hard-coding; the price printed on the ticket is the gross price.
3. `POST /invoices/drafts` with `date` = today, `currency: "DKK"`, customer, `recipient` copied from the
   customer (name, address, zip, city, `ean`, `vatZone`), `layout` and `paymentTerms` = agreement defaults from
   `GET /self` (`settings.defaultPaymentTerm`) or the bookkeeper's choice, `references.other` = the order
   reference `PF-<år>-<præfiks>-<nr>`, `notes.heading` = "Personaleforeningsbilletter TimeWinder <år>",
   `notes.textLine1` = barcodes first–last per variant, lines per variant with `product`, `description`,
   `quantity`, `unitNetPrice`, `discountPercentage: 0` (discount already applied in the price).
4. Store `draftInvoiceNumber`; mail the bookkeeper the link `https://secure.e-conomic.com` → Salg → Fakturakladder
   plus the number, instead of the long manual-data mail. Booking and sending stay with the bookkeeper.
5. TEST orders never create drafts (or create them on a trial agreement only).

## Validation checklist

- [ ] Both tokens come from Key Vault references; nothing token-like in code, logs, audit, or client responses.
- [ ] The grant token was obtained by an agreement administrator via the InstallationURL and stored as
      `ECONOMIC-AGREEMENT-GRANT-TOKEN`; the app role covers the endpoints used (Sales/SuperUser for invoices).
- [ ] Writes are drafts; booking/sending is a human step unless the owner decided otherwise.
- [ ] Every write has an `Idempotency-Key`; the created number is stored on the originating record.
- [ ] Mapping is a pure, tested module; e-conomic field names appear only in client + mapping.
- [ ] Required fields present: date, currency, customer, layout, paymentTerms, recipient.name,
      recipient.vatZone; lines with product also have quantity + description; no `null`s sent.
- [ ] VAT handling (vatZone, product group, net vs gross price) confirmed with the bookkeeper.
- [ ] Failure path logs e-conomic `logId`, surfaces a Danish message, and never retries a non-idempotent call
      without the same key.
- [ ] `outboundSuppressed()` guard present; no live e-conomic calls in tests or Store Review Mode.
