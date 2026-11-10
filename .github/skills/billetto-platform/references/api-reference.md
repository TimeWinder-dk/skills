# Billetto organiser API v3 and webhooks — working reference

Verified 2026-10-25 against https://api.billetto.com (reference pages are served as Markdown at
`/reference/<page>.md`; the index is `https://api.billetto.com/llms.txt`). Re-check field names against the
live reference before shipping; the docs carry several API versions side by side.

## Contents

1. Base URL, auth, versioning
2. Rate limiting
3. Resources and endpoints
4. Attendee object (the one you will use most)
5. Webhooks and signature verification
6. Public event search
7. Known gaps

## 1. Base URL, auth, versioning

- Base: `https://billetto.dk/api/v3` (the host selects the country; `billetto.dk` is TimeWinder's).
- Auth: header `Api-Keypair: <ACCESS_KEY_ID>:<ACCESS_KEY_SECRET>` (colon-joined, plain text — verified
  live 2026-10-25; secret alone, id alone and base64 all return 401 `authentication_error` "Missing access
  key secret"). Keys are created under Administrer → Integrationer → Developers (docs:
  `api.billetto.com/docs/obtaining-an-api-key`, whose curl example uses the pair on `/public/events`); the
  user needs the team permission "fuld adgang til Zapier, API og webhooks" to see that area. TimeWinder's pair lives in Key Vault
  `tw-opshub-kv` as `BILLETTO-API-KEY-ID` and `BILLETTO-API-KEY-SECRET`. OAuth2 (authorization code) also
  exists with scopes such as `read_tickets` and `read_organizer_event`; server-to-server integrations use
  the key pair.
- Version pinning: header `Billetto-Version`. Organiser endpoints default to `2026-02-10` (older allowed
  values: 2025-12-16, 2024-10-29, 2023-12-21, 2021-01-01, 2020-06-08, 2020-06-01). Pin it explicitly in
  every client so a server-side default bump cannot change payload shapes under you.
- Responses: 200 object, 400 error, 429 rate limit. Lists are `{ object: "list", data: [], has_more,
  total, url, next_url, prev_url }`.
- Pagination: cursor based with `after` / `before` (attendee ids) and `limit` 0–100.
- Expansion: `expand=` comma list; prefix with `data.` on list endpoints, e.g.
  `expand=data.ticket_type,data.order,data.ticket_buyer`.

## 2. Rate limiting

Cost based: every request has a cost; the account has a replenishing budget. Response headers:
`X-Ratelimit-Cost`, `X-Ratelimit-Limit`, `X-Ratelimit-Remaining`, `X-Ratelimit-Retry-After` (ms). On 429 stop
all calls for at least one minute and until Retry-After has passed; going negative triggers an extended
cool-off. Guidance from Billetto: queue and stagger requests, cache, filter fields, prefer webhooks over polling.

## 3. Resources and endpoints

| Resource | Endpoints | Notes |
|---|---|---|
| Events | `GET /organiser/events`, `GET /organiser/events/{id}` | **live fields:** `id, name, currency, state (draft/published/completed/canceled), public (bool), online_event, kind (regular/scheduled), image_url, total_capacity, parent, public_url, starts_at, ends_at, published_at, created_at, updated_at, pending_cancellation, availability {available, status}, plan, venue, organization, location`. Not `title`/`startdate`/`private` as some reference pages say. `availability.available` is the only count the API exposes. |
| Ticket types | `GET /organiser/ticket_types?event={id}` (**not** `/organiser/events/{id}/ticket_types`, which is 404) | fields: `id, uuid, type, name, quantity, fee_included, vip, description, min/max_tickets_per_order, sales_period, sells_from, sells_to, vat_rate, price (øre), index, admission, pdf, donation, merchandise, addons, group, event`. `type` seen live: `FreeTicketType`, `FoodAndBeverageTicketType`, `BundleTicketType` (packages). Each package item is its own ticket type id. No sold/remaining counter; whether `quantity` is capacity or remaining must be verified after a real sale. |
| Attendees | `GET /organiser/attendees`, `GET /organiser/events/{id}/attendees`, `GET /organiser/attendees/{id}` | filters `email, name, newsletter_permission, updated_after, updated_before, state, type, subscription, recurring_subscription` |
| Orders | `GET /organiser/orders`, `GET /organiser/orders/{id}` | order lines are always quantity 1 |
| Ledger | `GET /organiser/ledger_entries`, `/{id}` | money movements, for reconciliation |
| BI report | `GET /organiser/events/{eventid}/bi_report/` (JSON) and "send to e-mail" variant | report schema not documented; inspect a live response |
| Target groups | `GET/DELETE /organiser/target_groups/{id}`, imports, members `POST/PATCH/DELETE /organiser/target_group_members` | audiences and unique codes |
| Campaigns | `GET /organiser/campaigns`, event campaigns | read-only |
| Webhooks | `GET/POST /organiser/webhooks`, `GET/PATCH/DELETE /organiser/webhooks/{id}`, `POST .../secret`, revoke secret | Billetto recommends the GUI |
| Public | `GET /public/events`, `GET /public/events/{id}`, basket for a visitor id | see section 6 |

## 4. Attendee object

One attendee = one ticket = one barcode. Key fields:

| Field | Meaning |
|---|---|
| `id` | attendee/ticket UUID — use as the stable ticket id in TimeWinder systems |
| `barcode` | string scanned at the gate |
| `state` | `available, cancel_by_admin, cancelled, door_sale, draft, failed, manually_generated, mass_generated, refunded, reserved, sold` |
| `type` | `ticket` or `membership` |
| `ticket_type` | id or expanded object (name, price…) — this is where "partout", "familie", "fredag" live |
| `price`, `fee`, `fee_included` | integers in øre |
| `name`, `email`, `phone_number`, address fields | personal data — minimise what you copy |
| `order`, `order_line`, `ticket_buyer` | ids or expanded objects |
| `event`, `events` | event id(s) |
| `scannings` | a **list object** `{ object: "list", data: [...], has_more, total }`, not a bare array. Each entry: `id, object: "scanning", scanned_at (UTC Z), terminal_name, event_id, mode` (`entrance` or `exit`). Verified live: entrance → exit → entrance on "Scanner 1". |
| `booking_question_responses[]` | `id, question, answer, required…` |
| `newsletter_permission` | boolean |
| `created_at`, `updated_at` | ISO 8601 |

Reading rules that follow from this:

- A valid admission is an attendee whose `state` is one of `sold, door_sale, manually_generated,
  mass_generated` and that is not later `cancelled/refunded`; always re-read `state`, never cache "valid".
- An **arrival** is a `scannings.data[]` entry with `mode = entrance`. Exit scans (`mode = exit`) come from
  the universal exit code and must not be counted as entries.
- `barcode` is a numeric string (e.g. 11 digits); `price`/`fee` are integers in øre; `order` is an id
  unless expanded. `booking_question_responses` is also a list object.
- Observed request costs (budget 10 000 per window): events list ≈ 84–111, 5 attendees with expand ≈ 196,
  3 orders ≈ 653, 2 ledger entries ≈ 590, campaigns ≈ 160, target groups ≈ 8, webhooks ≈ 4. Orders and
  ledger are expensive; attendees with `updated_after` are the cheap incremental path.
- `terminal_name` is the free-text device name staff chose in the scanner app. It is the join key to
  TimeWinder's entrance mapping, so standardise device names before the festival.
- Incremental sync: `updated_after=<last cursor>` on the event attendee list; a new scan updates the
  attendee, so it surfaces through `updated_at`.

## 5. Webhooks

Configure under Integrate → Developers (GUI recommended) or via the API. Endpoint requirements: accept
`POST` JSON, respond 2xx/3xx (204 or 202 is fine, body ignored); otherwise Billetto retries up to 15 times with
exponential back-off `(retry_count ** 4) + 15 + rand(30) * (retry_count + 1)` seconds.

Event types (16): `event.created, event.published, event.unpublished, event.completed, event.cancelled,
event.updated, order.completed, order.refunded, order.cancelled, order.updated, attendee.registered,
attendee.cancelled, attendee.updated, attendee.scanned, attendee.generated, ledger_entry.created`.

Signature: header `Billetto-Signature: t=<unix ts>,v1=<hex>`; `v1` = HMAC-SHA256 over the string
`"<t>.<raw request body>"` using the webhook secret (generated per webhook; rotate via the secret endpoints).
Verify with a constant-time compare and reject stale timestamps (pick a tolerance, e.g. 5 minutes). The payload
schema is not documented; log the first deliveries and build your parser from real payloads, keyed by the
`Billetto-Version` the webhook was created with.

Billetto's own words to TimeWinder (2026-10-19): the public API can fetch tickets, barcodes and scanning
status, and a webhook can notify when a ticket is scanned in Billetto; an external system cannot register a
scan back into Billetto.

## 6. Public event search

`GET /public/events/{id}` returns `id, state, title, description, url (with UTM), image_link,
availability (boolean), minimum_price {amount, currency}, product_type, categorisation, organizer,
organization, location {…coordinates}, startdate, enddate, headliners`. No ticket types, no counts, no sale
windows. **Verified live:** the key is enforced (401 "Missing access key id" without it), and an event that
is published but has `public: false` returns 404 "Event not found". All TimeWinder 2027 events were
`public: false` on 2026-10-25, so neither this endpoint nor Basket works for them until they are made public.

Publisher requirements if you list Billetto events on your own site: keep the full event URL including UTM
parameters, mark the page canonical to the Billetto event (`<link rel="canonical">` or `Link` header), and
respect the rate limit.

## 7. Errors, expansion and the documentation map

- Error body: `{ "error": { "message": "…", "type": "invalid_request_error" } }` (other properties may
  appear); 4xx = your request (auth, missing parameter, rate limit), 5xx = Billetto. Only one `type` is
  documented, so branch on HTTP status, not on `type`.
- `expand=`: nested with dots (`event.organization`), list items with the `data.` prefix
  (`data.ticket_type`), several separated by commas, maximum depth four. Deep expansions on list requests
  are slow; expand only what the mapping needs.
- Versioning page (docs/versioning) still describes `v2021-01-01` as "current", while the organiser
  reference pages default to `2026-02-10`. The two pages disagree, which is exactly why every client pins
  `Billetto-Version` explicitly. A webhook is created with the then-current version and can be edited later.
- `api.billetto.com/reference/general-documentation-guides` and `docs/getting-started` are placeholder
  pages ("set up the welcome page…") with no content beyond a link; do not expect auth or scope details
  there. The real material is the per-endpoint reference pages, `docs/rate-limiting`,
  `docs/expanding-resources`, `docs/webhook-overview`, `docs/webhook-signatures`, `docs/errors` and the
  index `llms.txt`.
- Partnership and API access questions: support@billetto.com; developer contact in the spec: dev@billetto.com.

## 8. Verified live on 2026-10-25 (TimeWinder key)

Works (read): events, ticket types via `?event=`, attendees per event with `expand=data.ticket_type` and
`updated_after`, single attendee, orders with `expand=data.order_lines` (`applied_campaigns`, `used_codes`,
`payment_fees`, `billetto_fees`, `payout`, `sales_channel`, `terminal_name`, `manage_url`), ledger entries
(`entry_type` e.g. `ORDER_REVENUE`, `purchase_terminal`, `payment_gateway`, `amount`, `vat`), target groups
(`kind: static`, `segments`) and members (each with its unique `code`, `quantity`, `limit`), campaigns with
`conditions` (`type: time`) and `effects` (`percentage-discount-ticket-type-price` …), webhooks list, both
`Billetto-Version` 2021-01-01 and 2026-02-10.

Does not work / returns 404: `POST …/attendees/{id}/scan`, `PATCH …/attendees/{id}`, `POST
/organiser/scannings` (no check-in write-back, as Billetto said), `GET /organiser/events/{id}/bi_report[/]`
on both the festival and the camping event (404 — plan-gated, event-state-gated or a different path; ask
Billetto), `GET /organiser/events/{id}/ticket_types` (use the query-parameter form), `/public/events/{id}`
for non-public events.

## 8b. Write surface (verified 2026-10-25 with empty-body probes on the test event; nothing was created)

The organiser API can write exactly three things:

| Write | Endpoint | Notes |
|---|---|---|
| Target group member | `POST /organiser/target_group_members?target_group=<uuid>&code=<code>&quantity=1[&limit=…&space_identifier=…&category_key=…]` | parameters are **query** parameters, not a JSON body; 201 returns the member; 409 = code already in use; `PATCH`/`DELETE /organiser/target_group_members/{id}` exist. **`quantity` = total tickets the code can buy** (checkout shows `CODE 0/<quantity>`); `PATCH …/{id}?quantity=10&target_group=<uuid>` changes it (200, verified 2026-11-05). Cost ≈ 45–69 per POST/PATCH |
| Target group import | `POST /organiser/target_groups/{id}/imports` (see "Create a target group import") | bulk variant of the above |
| Webhooks | `POST /organiser/webhooks`, `PATCH`, `DELETE`, secret generate/revoke | with key-pair auth an empty POST returned **500** `undefined method 'application_id' for nil` — webhook creation appears to need an OAuth application; create webhooks in the GUI (Integrate → Developers) as Billetto recommends |

Everything else is read-only. Probed and **404**: `POST /organiser/orders`, `POST /organiser/events/{id}/orders`,
`POST /organiser/attendees`, `POST /organiser/events/{id}/attendees[/generate]`, `…/generate_tickets`,
`…/tickets`, baskets (organiser and public), invitations, `POST /organiser/events`, `PATCH
/organiser/events/{id}`, `PATCH /organiser/ticket_types/{id}`, box-office orders. So:

- **You cannot order, generate or send a ticket through the API.** "Generer billetter" (free tickets
  e-mailed to sponsors/exhibitors, max 20 per batch, counts towards capacity) is a dashboard feature only
  (Administrer event → Deltagere → Billethåndtering → Generer billetter).
- The automatable equivalent is **a unique code the recipient redeems**: create a target group member with
  a fresh code (the "Udstillere 2027" group already has 250), mail the recipient the event link plus the
  code, and let them claim the 0 kr "Udstillerbillet" through the access-code campaign. Billetto then sends
  the ticket. Redemption is visible on the order (`order_lines[].used_codes`) and via `attendee.registered`
  / `order.completed` webhooks, so an approval flow can close its own loop.
- **Personal one-click link (verified 2026-11-05, H-AA):** `https://billetto.dk/e/<slug>-<eventId>/select/<CODE>`
  opens the checkout with the code already applied (Billetto's own route `e/:slug-:id/select/:code?`; the
  page POSTs `/api/ticket_buyer/events/<id>/offers` with `code`). A query parameter (`/e/...?code=X`) is
  **ignored**, and `/ticket_buyer/offers/<uuid>` links are one-off checkout sessions — never send those.
  With `quantity=1` per member the link is effectively personal (one purchase per code). Widget variant:
  `embed/:slug-:id/tickets/:code?` (untested).

## 8c. All API versions compared (live, 2026-10-25)

`Billetto-Version` accepts `2026-02-10, 2025-12-16, 2024-10-29, 2023-12-21, 2021-01-01, 2020-06-08,
2020-06-01`; all seven answer 200 on events, attendees, ticket types and orders with TimeWinder's key. The
docs' `-1`/`-2` page variants are the same v3 endpoints for older/newer versions, not separate APIs. The only
changelog entry is "End of year update – Booking questions".

| Version | Difference vs the next newer version |
|---|---|
| 2026-02-10 (default) | events gain `pending_cancellation` |
| 2025-12-16 | events: `image_url`, `plan` (singular) instead of `plans`; ticket types lose `prices[]` |
| 2024-10-29 | ticket types carry `prices[]`; events `plans` |
| 2023-12-21 / 2021-01-01 | identical to 2024-10-29 |
| 2020-06-08 / 2020-06-01 | events embed `ticket_types`, `description(_html)`, `tags`, `category`, `host`, `manage_url`, `gallery_items`; attendees have `seat_label`, `events` but no `ticket_buyer`, `event`, `type`, `space`, `subscription`; orders lack `payment_fees`, `billetto_fees`, `payout`, `subtotal` |

`scannings` has the same list-object shape in every version. **No version adds a write endpoint**: orders,
attendees, tickets, ticket generator, ticket types and events are 404 for POST in all seven versions.
`/api/v1`, `/api/v2`, `/api/v4` do not exist. Pin `2026-02-10` (or `2024-10-29` if per-type `prices[]` is
needed).

`POST https://billetto.dk/api/orders` (no version prefix) answers 400 with a JSON:API error
"Var ikke i stand til behandle din ordre": that is the website's own internal checkout endpoint, not the
organiser API. It is undocumented, not covered by the API terms, and would create real orders — do not build
on it unless Billetto offers it officially as a partner checkout API.

## 9. Known gaps (ask dev@billetto.com or test live)

- BI report JSON schema and why the endpoint 404s for TimeWinder's events.
- Webhook payload schema and whether `attendee.scanned` carries the scan log entry or only the attendee id
  (no webhook configured yet; register one against a test endpoint and log the first deliveries).
- Semantics of ticket type `quantity` (capacity vs remaining) and of event `availability.available`.
- Whether the public endpoint and Basket work the moment `public` is flipped, or only after Billetto's
  marketplace listing is approved.
