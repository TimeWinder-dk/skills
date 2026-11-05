# Billetto embeds on timewinder.dk — what, where, how

Verified 2026-10-25 (support.billetto.com widget article, go.billetto.com/sell/checkout/widgets, live site).

## The site

- WordPress 7.1.3, theme **Inspiro** (WPZOOM), blocks: Kadence (+Pro), Essential Blocks, VK Blocks,
  Flexible Table Block. Plugins that matter: AIOSEO, Site Kit (GA4), GTranslate, WP Super Cache,
  Redirection, All-In-One Security, WP Smush, Eventin (demo content, to be removed).
- Hosting: webtohosting.dk (Apache). WordPress.org-style install, so custom scripts in blocks are allowed
  (WordPress.com would not be).
- Existing test page: `/test-billet-test/` ("TEST – Billet – TEST") with a plain link to the Nostalgi
  Camping 2027 event, two leftover widget labels and no live embed.

## Widget types and where their code is

| Widget | Where the snippet lives | Behaviour |
|---|---|---|
| Single-button | Event → Marketing → Widget | Button opens Billetto popup checkout on the site ("Sell directly on your website" on by default; off = redirect to event page) |
| Single-event (line style) | Event → Marketing → Widget | Lists ticket names, types, prices + "Get tickets" |
| Recurring-event calendar | Event → Marketing → Widget | Not relevant for a festival |
| Account widget | Sell → Widget | All events as calendar / list / button grid; updates itself when events are added; option "include private events" |

Snippet shape (replace the domain with `billetto.dk`):

```html
<script src="https://billetto.dk/embed.js"></script>
<!-- the generated element from My Billetto goes here; do not hand-edit its data attributes -->
```

Custom button alternative (plain link, no script):

```html
<a href="https://billetto.dk/e/e-2026708/select?color=1f4e79" class="tw-btn">Køb billetter</a>
```

Use the Smart Link URL from Rabatter & kampagner instead of the raw event URL when you need per-placement
reporting.

## Placement in WordPress

- Put the snippet in a **Custom HTML** block (or Kadence Advanced Text with "raw HTML"), never in the
  theme's `functions.php` or header: theme updates and the Inspiro customizer will not preserve it, and
  editors cannot see it.
- The block editor preview does not render the widget. Check the live page in a private window after
  purging WP Super Cache.
- One `embed.js` per page is enough even with several widgets.
- Keep the widget out of GTranslate's translated DOM if the layout breaks; the widget language is set in
  My Billetto, not by the page.

## Appearance

Light/dark theme, hex colours for buttons and elements, font choice, hide event image, white-label (remove
Billetto branding). If a colour is unreadable Billetto swaps to its default blue — pick a dark TimeWinder
colour with enough contrast against white.

## Cookies, headers, payments

- `_billetto_session`: essential cookie for the session and embedded checkout. List it in the consent tool as
  necessary; without it the popup checkout cannot complete.
- Blank new tabs from a widget usually mean `Cross-Origin-Opener-Policy` or `X-Frame-Options` headers are
  too strict. Check All-In-One Security's header settings and the host's defaults before changing anything;
  prefer relaxing on the sales page only.
- Apple Pay does not work inside the widget on external sites (Apple policy). The event page on
  billetto.dk has it, which is another argument for the button-to-event-page default.
- Basket (entry + camping in one order) never works inside a widget. If the page offers camping, link to
  the event page.

## Chat widget

```html
<script src="https://billetto.dk/chat_widget.js?token=<account token>"></script>
```

Tested March 2026 by Martin: it renders bottom-right and resists repositioning. Knowledge base = crawl of
the site URL plus selected Billetto events; no document upload. Treat the token as semi-public (it is in page
source anyway) but do not reuse it elsewhere.

## Things that must never be in WordPress

- The organiser API key (`Api-Keypair`) or any webhook secret.
- Direct calls to `billetto.dk/api/v3/organiser/*` from the browser.
- A second checkout (Eventin, WooCommerce) for the same tickets.
