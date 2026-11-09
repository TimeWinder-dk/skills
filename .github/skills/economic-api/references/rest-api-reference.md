# e-conomic REST API — working reference

Verified 2026-11-09 against `https://restdocs.e-conomic.com/`, the JSON schemas under
`https://restapi.e-conomic.com/schema/<resource>.<verb>.schema.json` and live GETs on the demo agreement
(`?demo=true`). The newer OpenAPI family at `https://apis.e-conomic.com/` (customers, products, quote-to-cash,
webhooks, documents …) was **not** verified (the page is a JS app); the legacy REST API below is what this skill
covers and what e-conomic still documents as the invoicing API.

## Contents

1. Base URL, headers, demo
2. Getting the AgreementGrantToken
3. Conventions: pagination, filtering, sorting, nulling, idempotency, errors
4. Endpoints that matter
5. Schemas: customer, draft invoice, booking
6. Master data snapshot (demo agreement)
7. Roles and permissions
8. Webhooks
9. Known gaps / to verify on the live agreement

## 1. Base URL, headers, demo

- Base: `https://restapi.e-conomic.com` — `GET /` lists every endpoint with its stability label.
- Headers on **every** call:

```
X-AppSecretToken: <AppSecretToken of the app>          (Key Vault tw-opshub-kv / ECONOMIC-APP-SECRET-TOKEN)
X-AgreementGrantToken: <grant token of the agreement>  (Key Vault tw-opshub-kv / ECONOMIC-AGREEMENT-GRANT-TOKEN)
Content-Type: application/json
Idempotency-Key: <your unique id>                       (writes only; forbidden on GET)
```

- Smoke test: `GET /self` → agreement number, company, user, `application` (appNumber, name, appPublicToken,
  requiredRoles), `settings.baseCurrency`, `settings.defaultPaymentTerm`, `canSendElectronicInvoice`, `modules`.
- Demo: both token headers = `demo`, or append `?demo=true`. **GET only.** Demo agreement 1583064 ("Demo
  Company"), base currency DKK, default payment term `1`.
- Status codes: 200/201 (201 with `Location` on create), 204 on delete, 400 validation, 401 bad/revoked tokens,
  403 app role lacks access, 404, 405, 415 (wrong content type), 500, 501.

## 2. Getting the AgreementGrantToken

1. Developer agreement → *Apps* → the app (TimeWinder: **Timewinderudtræk**) → *tokens*: shows
   **AppPublicToken**, **AppSecretToken** (reset-able; shown once), **RedirectURL**, **InstallationURL**
   `https://secure.e-conomic.com/secure/api1/requestaccess.aspx?appPublicToken=<AppPublicToken>` (optionally
   `&locale=da-DK`).
2. An **administrator of the accounting agreement** logs in to e-conomic, clicks *Administrer* on the agreement,
   then opens the InstallationURL and accepts. The end user must hold the permissions the app's role needs.
3. The **AgreementGrantToken** is either shown on screen (copy it to Key Vault) or, when a RedirectURL is set,
   the browser is redirected to it with `?token=<grant token>`. The redirect endpoint must be public and should
   confirm success to the user.
4. Expiry/revocation are not documented; treat the token as long-lived and revocable by the agreement admin
   (apps list in e-conomic). Re-run the installation to get a new one.
5. Trial agreements for rehearsing writes: `https://www.e-conomic.dk/regnskabsprogram/demo` (blank, 14 days) or
   `…/demo-alle` (with data). Extension: `api@e-conomic.com`.

## 3. Conventions

**Pagination** — `?pagesize=N` (default 20, max 1000) and `?skippages=K` (zero-based). Collections return
`{ collection: [...], pagination: { skipPages, pageSize, maxPageSizeAllowed, results, resultsWithoutFilter,
firstPage, nextPage?, lastPage }, self }`.

**Filtering** — `?filter=<property>$<op>:<value>`; operators `$eq $ne $gt $gte $lt $lte $like $in $nin $null`,
combinators `$and: $or:` with parentheses. Only properties with `"filterable": true` in the schema; others ⇒ 400.
String matching is case-insensitive; `*` is the wildcard for `$like`; `$in` takes `[1,2,3]` (numeric, max 200).
URL-encode the whole filter; escape literal `$ ( ) *` with a leading `$`. Examples:

```
/customers?filter=corporateIdentificationNumber$eq:12345678
/customers?filter=name$like:*personaleforening*
/invoices/drafts?filter=customer.customerNumber$eq:42
/customers?filter=customerNumber$in:[2,5,7]
```

**Sorting** — `?sort=name`, `?sort=-name,age`, `~name` for alphabetical sort of numeric values; only
`"sortable": true` properties.

**Nulling** — omit the property to clear it; explicit `null` generally fails validation.

**References** — nested objects are identified by number **or** `self` URI (`{ "customerNumber": 1 }` or
`{ "self": "https://restapi.e-conomic.com/customers/1" }`). Changing a reference on PUT does not refresh the
copied data (e.g. a draft's `recipient` keeps the old name/address).

**Idempotency** — `Idempotency-Key` on POST/PUT; same key within one hour ⇒ cached response,
`X-ResultFromCache: true`.

**Errors** — validation:

```json
{
  "message": "Validation error.",
  "developerHint": "Inspect validation errors and correct your request.",
  "logId": "…",
  "httpStatusCode": 400,
  "errors": {
    "customer": { "errors": [ { "errorCode": "E06000", "message": "customer does not exist.", "value": "987987987",
                                "developerHint": "Find a list of resources at https://restapi.e-conomic.com/customers." } ] },
    "lines": { "items": [ { "arrayIndex": 1, "quantity": { "errors": [ { "errorCode": "E04300", "message": "quantity is required when product is set." } ] } } ] }
  },
  "logTime": "2015-03-12T16:44:56"
}
```

Errors mirror the payload; line errors carry `arrayIndex`. Log `logId` + `logTime` for support.

**Versioning** — none. Root = latest. Stability labels on `GET /`.

**Rate limiting** — not documented (no numbers, no headers, no 429 in the status table, checked 2026-11-09). Treat
429/503 as "stop and retry later with the same Idempotency-Key".

## 4. Endpoints that matter

| Resource | Verbs | Notes |
|---|---|---|
| `/self` | GET | Agreement, company, app, defaults (`settings.defaultPaymentTerm`), modules |
| `/customers`, `/customers/:customerNumber` | GET, POST, PUT, PATCH?, DELETE | Filter on `corporateIdentificationNumber`, `email`, `name`; sub-resources `/contacts`, `/delivery-locations`, `/invoices/{drafts,booked}`, `/totals`, `/templates/invoice` (a ready draft body for that customer) |
| `/customers/:n/contacts` | GET, POST, PUT, DELETE | Contact persons; needed for `recipient.attention` and e-mail sending rules |
| `/customer-groups` | GET, POST, PUT, DELETE | `customerGroupNumber`, `name`, `account.accountNumber` (debtor account) |
| `/products`, `/products/:productNumber` | GET, POST, PUT, DELETE | `productNumber` ≤ 25 chars (string), `productGroup` decides sales account + VAT, `salesPrice`, `unit`, `barCode`, `barred` |
| `/product-groups`, `/units` | GET (+ write) | Master data |
| `/payment-terms`, `/vat-zones`, `/layouts` | GET | Master data; referenced by number on customers and invoices |
| `/invoices` | GET | Links to `drafts`, `booked`, `sent`, `paid`, `unpaid`, `overdue`, `notDue`, `totals` |
| `/invoices/drafts` | GET, POST, DELETE(all, filterable) | Draft = not yet booked; default sort `draftInvoiceNumber` |
| `/invoices/drafts/:n` | GET, PUT, DELETE | PUT requires `recipient` + `recipient.vatZone`; `/lines` POST adds lines; `/pdf`; `/attachment` (PDF ≤ 9 MB); `/templates/booking-instructions` = body for booking |
| `/invoices/booked` | GET, POST | POST books a draft (see §5); `/invoices/booked/:n` GET, `/pdf` |
| `/invoices/totals/...` | GET | Counts and net amounts (drafts, booked, unpaid, overdue), filter by customer/year/period |
| `/orders`, `/quotes` | GET, POST, PUT, DELETE | Same shape as invoices; `…/templates/upgrade-instructions/draftInvoice` turns an order into a draft invoice body |
| `/journals/:n/vouchers` | GET, POST | Manual postings (Bookkeeping role) — not used for tickets |
| `/app-roles`, `/accounting-years`, `/accounts`, `/currencies`, `/employees`, `/suppliers`, `/webhooks` | GET (+) | Reference |

## 5. Schemas

### Customer — `POST /customers` (`schema/customers.post.schema.json`)

Required: **`name`** (1–255), **`currency`** (3), **`customerGroup.customerGroupNumber`**,
**`vatZone.vatZoneNumber`**, **`paymentTerms.paymentTermsNumber`**.
Optional: `customerNumber` (1–999999999, assigned when omitted), `address` (510), `zip` (30), `city` (50),
`country` (50), `email` (255), `telephoneAndFaxNumber` (255), `mobilePhone` (50), `corporateIdentificationNumber`
(CVR, 40), `pNumber` (10), `vatNumber` (20), `ean` (13), `publicEntryNumber` (50), `website` (255), `creditLimit`,
`barred`, `layout.layoutNumber`, `salesPerson.employeeNumber`, `priceGroup`, `eInvoicingDisabledByDefault`
(PATCH only). Read-only on GET: `balance`, `dueAmount`, `lastUpdated`, links (`contacts`, `templates.invoice`,
`invoices.drafts/booked`, `totals`).

```json
{
  "name": "Novo Nordisk Personaleforening",
  "currency": "DKK",
  "customerGroup": { "customerGroupNumber": 1 },
  "vatZone": { "vatZoneNumber": 1 },
  "paymentTerms": { "paymentTermsNumber": 4 },
  "email": "pf@example.dk",
  "corporateIdentificationNumber": "12345678",
  "ean": "5790000000000",
  "address": "Vejnavn 1", "zip": "2880", "city": "Bagsværd", "country": "Danmark"
}
```

### Draft invoice — `POST /invoices/drafts` (`schema/invoices.drafts.post.schema.json`, draft-03)

Required: **`date`** (`YYYY-MM-DD`), **`currency`**, **`customer.customerNumber`**, **`layout.layoutNumber`**,
**`paymentTerms.paymentTermsNumber`**, **`recipient.name`** (250), **`recipient.vatZone.vatZoneNumber`**.
`dueDate` only when the payment term type is `dueDate`. Read-only: `netAmount`, `vatAmount`, `grossAmount`,
`*InBaseCurrency`, `roundingAmount`, `marginPercentage`, `pdf.download`.

| Block | Properties |
|---|---|
| `recipient` | `name`*, `address` (250), `zip` (50), `city` (250), `country` (50), `ean` (13), `publicEntryNumber` (40), `cvr` (40, list views), `attention.customerContactNumber`, `vatZone.vatZoneNumber`*, `mobilePhone`, `nemHandelType` (`ean` / `corporateIdentificationNumber` / `pNumber` / `peppol`) |
| `references` | `customerContact.customerContactNumber`, `salesPerson.employeeNumber`, `vendorReference.employeeNumber`, `other` (250, free text — use for the order reference) |
| `notes` | `heading` (250), `textLine1` (1000), `textLine2` (1000) |
| `delivery` | `address` (255), `zip` (30), `city` (50), `country` (50), `deliveryTerms` (100), `deliveryDate` |
| `deliveryLocation`, `project` | by number |
| `lines[]` | `lineNumber`, `sortKey`, `product.productNumber` (25), `description` (2500; required when a product is set), `quantity` (2 dec), `unitNetPrice` (2 dec), `discountPercentage`, `unit.unitNumber`, `unitCostPrice`, `accrual.startDate/endDate`, `departmentalDistribution` |

```json
{
  "date": "2027-02-01",
  "currency": "DKK",
  "customer": { "customerNumber": 1234 },
  "layout": { "layoutNumber": 21 },
  "paymentTerms": { "paymentTermsNumber": 4 },
  "recipient": { "name": "Novo Nordisk Personaleforening", "address": "Vejnavn 1", "zip": "2880", "city": "Bagsværd",
                 "ean": "5790000000000", "vatZone": { "vatZoneNumber": 1 } },
  "references": { "other": "PF-2027-pnov-003" },
  "notes": { "heading": "Personaleforeningsbilletter TimeWinder 2027",
             "textLine1": "Voksen pnov0002510–pnov0002519 · Barn pnov0002520–pnov0002525 · Familie pnov0002526–pnov0002528" },
  "lines": [
    { "lineNumber": 1, "product": { "productNumber": "PF-VOK" }, "description": "Voksen lørdag/søndag, TimeWinder 2027 (20 % rabat)", "quantity": 10, "unitNetPrice": 160.00, "unit": { "unitNumber": 1 } },
    { "lineNumber": 2, "product": { "productNumber": "PF-BRN" }, "description": "Barn lørdag/søndag, TimeWinder 2027 (20 % rabat)",   "quantity": 6,  "unitNetPrice": 80.00,  "unit": { "unitNumber": 1 } },
    { "lineNumber": 3, "product": { "productNumber": "PF-FAM" }, "description": "Familie lørdag/søndag, TimeWinder 2027 (20 % rabat)", "quantity": 3,  "unitNetPrice": 400.00, "unit": { "unitNumber": 1 } }
  ]
}
```

Response `201` = the draft with `draftInvoiceNumber`, computed totals, `pdf.download`, `templates.bookingInstructions`.
`unitNetPrice` is **net of VAT** — whether tickets carry VAT is a bookkeeper decision (see SKILL recipe step 2).

### Book a draft — `POST /invoices/booked` (`schema/invoices.booked.post.schema.json`)

```json
{ "draftInvoice": { "draftInvoiceNumber": 123 }, "bookWithNumber": 456, "sendBy": "Email" }
```

All optional at schema level; in practice `draftInvoice` is needed. `bookWithNumber` forces the invoice number.
`sendBy` ∈ `none` | `EAN` | `Email` (omitted ⇒ not sent). Alternative `PUT /invoices/booked/:bookWithNumber`.
E-mail sending needs the agreement contact e-mail + a recipient on the customer (default invoice e-mail, a
contact with *Receive invoices*, or the invoice's AttEmail/YourRefEmail); EAN needs CVR on the agreement, EAN on
the customer and contact persons on both (Danish, paying agreements only).

### Booked invoice — `GET /invoices/booked/:bookedInvoiceNumber` (verified on demo 2026-11-09)

Top level: `bookedInvoiceNumber`, `orderNumber`, `date`, `dueDate`, `currency`, `exchangeRate`, `netAmount`,
`netAmountInBaseCurrency`, `grossAmount`, `grossAmountInBaseCurrency`, `vatAmount`, `roundingAmount`,
**`remainder`**, **`remainderInBaseCurrency`** (outstanding balance; `0.00` = fully paid), `paymentTerms`,
`customer`, `recipient`, `deliveryLocation`, `delivery`, `notes`, `references` (`customerContact`, `salesPerson`,
`other`), `layout`, `project`, `lines`, `pdf.download`, `sent`, `self`.

Filterable/sortable on the collection (schema): `bookedInvoiceNumber`, `date`, `currency`, `exchangeRate`,
`customer.customerNumber`, **`references.other`** — live check `GET /invoices/booked?filter=references.other$eq:x`
returned 200 with an empty collection (not 400). **Not** filterable: `remainder`, `grossAmount`, `dueDate` —
fetch the single invoice and compare in code.

Views: `/invoices/paid` (remainder 0), `/invoices/unpaid`, `/invoices/overdue` (unpaid past `dueDate`),
`/invoices/notDue`, `/invoices/sent`; same item shape.

### PDF

`GET /invoices/drafts/:n/pdf` and `GET /invoices/booked/:n/pdf` return the PDF (also linked as `pdf.download`).

## 6. Master data snapshot (demo agreement, 2026-11-09)

Numbers differ on TimeWinder's agreement — read them from the live API before mapping.

- **VAT zones:** 1 Domestic, 2 EU, 3 Abroad, 4 Domestic exempt VAT (+ per-country zones 8…).
- **Payment term types:** `net`, `invoiceMonth`, `paidInCash`, `prepaid`, `dueDate`, `factoring`,
  `invoiceWeekStartingSunday`, `invoiceWeekStartingMonday`, `creditcard`, `avtaleGiro`; `daysOfCredit` only when
  the type is not `dueDate`. Demo: 1 "Netto 8 dage", 4 "Netto 14 dage", 2 "Løbende måned 30 dage".
- **Layouts:** e.g. 21 "DK. std. m. bankoplys. 1.8", 22 "DK. std. m. girokort 1.8" (`layoutNumber`, `name`).
- **Units:** 1 stk., 2 timer, … (`unitNumber`, `name`).
- **Product groups:** `productGroupNumber`, `name` ("Varer m/moms", "Ydelser u/moms" …), `salesAccounts` or
  `accrual`, `inventoryEnabled`.
- **Customer groups:** `customerGroupNumber`, `name`, `account.accountNumber`.

## 7. Roles and permissions (`e-conomic.com/developer/permissions`)

| Endpoint | SuperUser | Bookkeeping | Sales | Project Employee |
|---|---|---|---|---|
| `/customers`, `/customers/:n/contacts`, `/customer-groups` | ✓ | | ✓ | |
| `/invoices/drafts`, `/invoices/booked` | ✓ | | ✓ | |
| `/orders` | ✓ | | ✓ | |
| `/products` | ✓ | | ✓ | ✓ |
| `/product-groups`, `/employees` | ✓ | | ✓ | |
| `/suppliers`, `/accounts`, `/journals/:n/vouchers` | ✓ | ✓ | | |
| `/self` | ✓ | | ✓ | |

The role is fixed on the app in the developer portal; the end user granting access must also hold the matching
rights. A missing right ⇒ `403`.

## 8. Webhooks (`e-conomic.com/developer/working-with-web-hooks`)

Managed through the Webhooks API (create/retrieve/update/delete; an endpoint lists supported event types).
Delivery: HTTP POST per changed resource (bulk changes ⇒ many posts), usually prompt but not guaranteed.
Retries on connection error / non-2xx: up to 5, after 1 s, 20 s, 1 min, 5 min, 20 min; then logged as failed.
**> 250 failures to one URL in 24 h disables the webhook** and events are lost until re-enabled — so queue
first, process later, and keep a nightly reconciliation job. Payload shape and callback verification are in the
OpenAPI docs (not verified here). Not needed for the ticket invoicing v1.

## 9. Known gaps / verify on the live agreement

- `references.other` is filterable on `/invoices/booked` (confirmed); confirm the same on `/invoices/drafts`
  on the live agreement (needed for the duplicate-draft check).
- Customer **contacts** POST schema (`/customers/:n/contacts`) — required fields and the e-mail notification
  flags were not retrievable (schema URL 404); read `GET /customers/:n/contacts` on the live agreement.
- TimeWinder's actual numbers: customer group for personaleforeninger, VAT zone for tickets, payment terms,
  layout, the three product numbers, the app's role (Sales vs SuperUser).
- Rate limits: none published; observe real behaviour and note it here.
- OpenAPI family (`apis.e-conomic.com`): base paths, auth and whether invoicing exists there; only relevant if
  the legacy REST API is deprecated for invoices.
