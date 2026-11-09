# TimeWinder's e-conomic app and agreement — snapshot 2026-11-09

Dated snapshot; update it the day something changes (and write a log entry in Operations Hub `docs/ai/log/`).

## The app (owner decision 2026-11-09: one app, role SuperUser)

One app serves both purposes — ticket invoicing for personaleforeninger (EPIC-108) **and** reporting extracts of the
whole set of books (chart of accounts, entries, income/expenses per account and period). Extracts need `/accounts`,
`/accounting-years/*/entries`, `/journals`, `/suppliers`, `/budgets`, which only **SuperUser** reaches, so the app
carries SuperUser. The broad role is compensated in code (see SKILL "Wiring": write allowlist, draft-only writes,
separate modules), not by a second app.

| Item | Value / where |
|---|---|
| Developer-portal app | **Timewinderudtræk** (created by the owner, tha@timewinder.dk, 2026-11-09), role **SuperUser** |
| AppPublicToken | Key Vault `tw-opshub-kv` → `ECONOMIC-APP-PUBLIC-TOKEN` (not secret; it is part of the InstallationURL) |
| AppSecretToken (`X-AppSecretToken`) | Key Vault `tw-opshub-kv` → `ECONOMIC-APP-SECRET-TOKEN`; reset in the portal if leaked, then update the secret (grants stay valid) |
| AgreementGrantToken (`X-AgreementGrantToken`) | Key Vault `tw-opshub-kv` → `ECONOMIC-AGREEMENT-GRANT-TOKEN` — **missing** |
| RedirectURL | empty (grant token is shown on screen after acceptance) |
| InstallationURL | `https://secure.e-conomic.com/secure/api1/requestaccess.aspx?appPublicToken=<ECONOMIC-APP-PUBLIC-TOKEN>` |

## The agreement grant — MISSING (2026-11-09)

Nothing can be called until an **administrator of TimeWinder's accounting agreement** (the bookkeeper or the
owner, logged in to the books — not the developer agreement) opens the InstallationURL, clicks *Administrer* on
the agreement first if asked, accepts the app and hands over the shown token. Store it as
`ECONOMIC-AGREEMENT-GRANT-TOKEN` in `tw-opshub-kv`, then smoke-test:

```
GET https://restapi.e-conomic.com/self
X-AppSecretToken: <secret>   X-AgreementGrantToken: <grant>
```

Record from the response: `agreementNumber`, `company.name`, `settings.baseCurrency`,
`settings.defaultPaymentTerm`, `canSendElectronicInvoice`, `application.requiredRoles`.

## Operations Hub wiring (planned, not built)

- App settings (Key Vault references in `infra/main.bicep`): `ECONOMIC_APP_SECRET_TOKEN`,
  `ECONOMIC_AGREEMENT_GRANT_TOKEN`; plain `ECONOMIC_BASE_URL` (default `https://restapi.e-conomic.com`). One client
  module; the ticket flow and the extract module import it with different, explicit endpoint allowlists.
- Client + mapping modules under `backend/src/services/economic/` (pattern: `sponsorTickets/sponsorFlowClient.ts`).
- First consumer: EPIC-108 personaleforeningsbilletter — draft invoice per order instead of (or in addition to) the
  mail to `bogholderi@timewinder.dk`. Decision pending (design D9).

## Master data to agree with the bookkeeper before the first draft

| Need | Why | Where it ends up |
|---|---|---|
| Customer group for personaleforeninger (`customerGroupNumber`) | required on `POST /customers` | app setting `ECONOMIC_CUSTOMER_GROUP` |
| VAT zone for ticket sales (`vatZoneNumber`: Domestic vs Domestic exempt) and product group | decides whether `unitNetPrice` is net of 25 % or equals the printed price | mapping config |
| Payment terms (`paymentTermsNumber`) and layout (`layoutNumber`) | required on every draft | from `GET /self` defaults or settings |
| Three products `PF-VOK`, `PF-BRN`, `PF-FAM` (names final with bookkeeper) | one line per ticket variant | created once in e-conomic, numbers in settings |
| Whether Ops Hub may **create customers** or only link existing ones (`EconomicKundenr` column on the SharePoint list) | avoids duplicate debtors | owner decision |
| Who books and sends (`sendBy`) | invoices leave e-conomic by e-mail/EAN only when booked | bookkeeper — Ops Hub stops at the draft |

## Spike plan (before any production code)

Owner decision 2026-11-09: the e-conomic integration is a **spike**. Time-box: one working day on the real
agreement (or a trial agreement if the bookkeeper prefers), scripted with plain `curl`/PowerShell from the
scratchpad, tokens read from Key Vault, nothing committed but the result.

| # | Question | How to answer | Exit evidence |
|---|---|---|---|
| S1 | Can we get the grant token, and which role does the app have? | Admin opens InstallationURL; `GET /self` | `application.requiredRoles`, agreement number, defaults recorded in this file |
| S2 | Which master data do we map? | `GET /customer-groups`, `/vat-zones`, `/payment-terms`, `/layouts`, `/product-groups`, `/units` | Numbers for personaleforeninger written here; VAT zone/product group confirmed by the bookkeeper |
| S3 | Can we create a customer and a draft with `references.other`? | `POST /customers` (test customer), `POST /invoices/drafts` with 3 lines, `Idempotency-Key`; repeat the POST with the same key | 201 + `draftInvoiceNumber`; the repeat returns the cached response (`X-ResultFromCache`) |
| S4 | Does the payment gate work end to end? | Bookkeeper books the test draft, registers a (test) payment; `GET /invoices/booked?filter=references.other$eq:<ref>` → `/invoices/booked/:n` | `remainder` goes from gross to 0.00; latency from bank registration to API visibility measured |
| S5 | Failure modes | Wrong grant token, Bookkeeping-role token, missing `description` on a product line, duplicate reference | 401 / 403 / 400 bodies with `logId` captured in the reference |
| S6 | Credit note behaviour | Bookkeeper credits the test invoice | What the API shows afterwards (`remainder`, new document) — documented |
| S7 | Clean-up | `DELETE /invoices/drafts/:n`, bar or delete the test customer | Agreement left clean; bookkeeper informed |

Decision after the spike: (a) Ops Hub creates drafts + polls payment (full automation), (b) bookkeeper creates
invoices by hand with the order reference in *Anden reference* and Ops Hub only polls payment, or (c) mail-only
(no API). Estimate the EPIC-108 build tasks only after this choice.

## Contacts

- Bookkeeping: shared mailbox `bogholderi@timewinder.dk` (confirmed by the owner 2026-11-09).
- e-conomic API support: `api@e-conomic.com`, phone 88 20 48 40.
