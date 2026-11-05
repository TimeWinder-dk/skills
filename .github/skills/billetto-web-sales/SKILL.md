---
name: billetto-web-sales
version: 1.0.0
releaseDate: 2026-10-25
description: |
  How to turn timewinder.dk (WordPress) into the primary Billetto sales channel: the buy-button + Smart Link
  pattern, when a widget is allowed and when it is not (Basket), the anatomy of a sales page (prices, family
  packages, camping cross-sell, practical info), tracking (UTM/Smart Links/GA4), campaigns, drops and codes,
  cookie consent and security headers, and what must NOT be embedded in WordPress. Use this skill whenever a
  task touches the website's ticket sales, "Køb billetter"-knap, salgsside, priser-siden, widgets, embed code,
  early bird/kampagner on the site, live "billetter tilbage" counters, the chat widget, or landing pages for
  udstillere/klubber/venner — even if Billetto is not named. Pair with billetto-platform (read first) and, for
  anything that calls an API, billetto-ops-hub.
owner: TimeWinder IT
lastReviewed: 2026-10-25
---

# Billetto Web Sales

The website's only job in ticketing is to get a visitor from "interested" to a completed Billetto order with
as little friction as possible, and to let TimeWinder measure which page did it. Billetto owns checkout,
payment, PDF tickets and refunds; WordPress owns persuasion, navigation and measurement.

## DO NOT USE FOR

- Changing prices, capacities or packages — that is done in My Billetto (see `billetto-platform`).
- Server-side integrations (webhooks, API clients) — `billetto-ops-hub` / `billetto-tw-tickets`.
- Scanning or gate flows — `billetto-own-ticket-app`.

## Required workflow

1. Read `billetto-platform` (limits: Basket is public-only and never inside a widget; Apple Pay is
   unavailable in the widget on external sites; the public API has no per-type availability).
2. Decide the sales entry pattern with the table below. Default for TimeWinder is **button → event page**.
3. Build or change the page with the blueprint in
   [references/sales-page-blueprint.md](references/sales-page-blueprint.md).
4. Embed only what [references/widget-embed.md](references/widget-embed.md) allows, in a Custom HTML or
   Kadence Advanced Text block, never in the theme files, and never with an API key.
5. Add measurement: a Smart Link per placement, UTM on every outbound Billetto link, a GA4 event on the
   click (Site Kit is installed).
6. Verify on the live page in a private window (WP Super Cache and the block editor preview both hide
   embeds), on phone width, and with cookie consent declined.

## Sales entry patterns

| Pattern | Use when | Why |
|---|---|---|
| **Buy button → Billetto event page (Smart Link)** | Entry + camping must be bought together (Basket), or any multi-event purchase | Basket lives only on billetto.dk. This is TimeWinder's chosen default. |
| Single-button widget (popup checkout on the site) | One event, one decision, e.g. a Friday concert page or an exhibitor page with a code | Keeps the visitor on the site; no Basket; Apple Pay missing |
| Account widget, list view, "no direct sales" | Overview of all events on the Priser page | Calendar view opens on today's date, not the festival; list is clearer for three events |
| Custom `<a>` to `/e/e-<id>/select` | Deep link straight into ticket selection, e.g. from a newsletter | Still a normal link; add UTM/Smart Link |

Never mix patterns for the same ticket on one page: one page, one primary call to action.

## Measurement rules

- Create one Smart Link (Rabatter & kampagner → Smart links & QR) per placement: forside-knap, priser-side,
  camping-side, nyhedsbrev, Facebook. Billetto cannot split sales retroactively, so do this before launch.
- Keep Billetto's UTM parameters on event URLs; add your own `utm_content` per placement only via Smart
  Link so the two systems agree.
- Fire a GA4 event (`select_content` or a custom `ticket_cta_click`) on every buy-button click so the site
  funnel can be compared with Billetto's sales report.
- A live counter ("X partout tilbage", "early bird slutter om…") must come from a cached endpoint in
  Operations Hub, never from the browser calling Billetto: the public endpoint has no counts and the
  organiser API needs the secret key.

## Conversion levers that exist in Billetto today

- **Drop Sales**: scheduled ticket releases. Use them against the known pattern that sales only rise in the
  last 14 days before Pinse: a March drop with a visible deadline gives a reason to buy early.
- **Early bird campaign** with an end date shown on the page.
- **Easy Discounts / access codes** for TimeWinders Venner, klubber, udstillere (250 unique codes already
  exist), sponsors and ambassadors; each code is also a report of who sold.
- **Waiting list** when a type is sold out (camping first).
- **Chat widget** (`billetto.dk/chat_widget.js?token=…`, GPT-4o, crawls the site): fine as a support
  layer; it renders bottom-right and cannot be moved easily. Do not rely on it for navigation.
- **Family packages** with one barcode per person: explain on the page that every family member gets a
  ticket, because the old "one ticket per family" expectation will confuse returning guests.

## Compliance and site hygiene

- Declare the `_billetto_session` cookie as necessary in the consent banner (checkout and popup depend on it).
- If widgets open blank tabs, check `Cross-Origin-Opener-Policy` and `X-Frame-Options` set by the theme or
  All-In-One Security before relaxing anything, and document what you changed.
- Remove or disable the Eventin plugin's demo content; the demo links and its Razorpay webhook route are
  live in the HTML today and have no business on a production site.
- Names and prices on the site must match the Billetto ticket type names and the price *including* the fee
  the buyer sees, or support mail will follow.
- Pages that list Billetto events via the API must be canonical to the Billetto event URL (publisher
  requirement).

## Validation checklist

- [ ] One primary CTA per page, pattern chosen deliberately (Basket needs → event page).
- [ ] Smart Link per placement + UTM intact + GA4 click event.
- [ ] Embed lives in a block, contains no secrets, verified live in a private window and on mobile.
- [ ] Cookie consent lists `_billetto_session`; security headers checked, not blindly relaxed.
- [ ] Prices/names on the page equal Billetto's buyer-facing values; family package explained.
- [ ] Any live counter comes from an Operations Hub endpoint with caching, not from the browser.
- [ ] Eventin demo content gone; no second ticket system visible to guests.
