# TimeWinder's Billetto account — state snapshot

Snapshot date: **2026-10-25**, assembled from the mail dialogue in tha@timewinder.dk (threads
"TimeWinders spørgsmål i forhold til nyt Billet håndterings system", "Billet scannings Hardware",
"Samarbejde mellem Billetto & TimeWinder", "Scanning af flere events på en gang", "Samle flere events i et
Widget billede. En PDF fil per billet. Depositum") and the live website. Update this file when the facts
change; do not treat it as live state.

## People

| Role | Name | Contact |
|---|---|---|
| TimeWinder setup owner | Carsten Olsen, IT-koordinator | co@timewinder.dk, +45 61 48 50 77 |
| TimeWinder IT / integrations | Thomas Haulund-Aadorf | tha@timewinder.dk |
| TimeWinder marketing (declined ticket graphics 2026-10-17 pending board plan) | Lene Pels Jørgensen | lenpel@timewinder.dk |
| TimeWinder web (placed the chat widget) | Martin Thorvaldsen | mth@timewinder.dk |
| Billetto Account Executive | Mikkel Sørensen | miso@billetto.com, +45 28 44 47 00 |
| Billetto CSM (DK) | Bengta Dahlsjö | bengta@billetto.com |
| Billetto support | | +45 71 99 38 82, support.billetto.com |

The organiser account's system mails go to the M365 group **"IT Systemer på tværs"**; Thomas was added as a
user on 2026-02-13.

## Source documents (SharePoint "IT Systemer på tværs", synced locally)

`…\Documents - IT Systemer på tværs\General\Billet systemer\`:

- `Spørgsmål til Billetudsteder.xlsx` (Feb 2026): the vendor comparison (Place2Book, Billetto, Understory,
  Billetexpressen, Billetsalg, Eventbilletten) with Billetto's written answers from 2025-01-20. Billetto
  answered **"Ja"** to "API: mulighed for køb og administration af billetter via egne systemer", **"Ja"** to
  "hente billetter direkte via API som BASE64/links (Power Automate)" and **"ja"** to a pre-purchase
  validation/approval flow — none of which the organiser API provides today. Fee sheet: Billetto 2,5 % +
  4,50 kr, example 200 kr ticket ≈ 10,77 kr fees incl. VAT vs ≈ 15,19 kr at Place2Book; yearly estimate ≈
  43 000 kr vs ≈ 60 700 kr.
- `Nyt billet system.docx` (Jan 2026): vendor notes and TimeWinder's requirement list; also Eventbilletten's
  offer (4 kr + 4 % + VAT, Zebra TC25-27, purchase-intent approval flow, 3-year deal) as the runner-up.
- `Billetto\Billetto billet oprettelse.docx` (Carsten, 2026-10-24): the setup runbook (event checklist,
  widget colours #056FDA/#FFFFFF, unique codes via Promover → Audiences → Generer koder → campaign with
  100 % discount, family package creation, booking questions, Multi Scanner 36 h) plus Carsten's open
  questions: hide an event id in the widget, "kun billetsælger" ticket, ticket types vanishing when a code
  is entered, replacement for the 0 kr queue-creating ticket, booking question "campingvogn" wrongly required
  on tent tickets (sent to Mikkel 2026-10-24, unanswered 2026-10-25), camping start date vs Multi Scanner.
  Contains a shared login for the sekretariat user — do not copy it anywhere.
- `Billetto\Billeder\`: candidate banner/hero images. `Place2Book\`: the old vendor's ticket backgrounds
  (2023–2025) and docs, useful as the baseline for the Billetto banner.

## Timeline

- 2025-01: question sheet answered by Billetto (fees offered 2,5 % + 4,50 kr + VAT). Decision parked until
  after Pinse 2025.
- 2025-12 to 2026-02: scanner hardware (used Zebra TC57 are fine; Billetto can lend a few scanners), POS
  pricing, collaboration confirmed, meeting 2026-02-13, account created, fee adjusted on the account, Adyen
  recommended over Stripe and chosen by TimeWinder (2026-04-08).
- 2026-03 to 2026-05: test events, widget experiments, chat widget script tested on the website. 2026 festival
  still ran on Place2Book; the three Billetto 2026 events were test shells (2, 1 and 3 "sales", 0 kr).
- 2026-10: real 2027 setup. Basket enabled (2026-10-16), family packages working (2026-10-15), Multi Scanner
  enabled (2026-10-19), deposit model confirmed by Billetto finance (2026-10-19), target group "Udstillere
  2027" with 250 unique codes linked to a campaign (2026-10-21, after a UI glitch).

## Events *(verified via the API 2026-10-25; all 2027 events were `public: false`)*

| Event | Id | State | Notes |
|---|---|---|---|
| TimeWinder Nostalgi-festival 2027 | 2026708 | published, not public | main entry event, 14–16 May 2027, `availability.available` 68 153 |
| Nostalgi Camping på TimeWinder 2027 | 2026700 | published, not public | available 665; types include Nostalgivogn/VW/Telt bundles (`BundleTicketType`) with partout adult |
| Camping på TimeWinder 2027 | 2028486 | published, not public | `total_capacity` 225 |
| Udstillerbillet 2027 | 2028490 | published, not public | available 2 970; codes via target group "Udstillere 2027" |
| Markedsboder på TimeWinder 2027 | 2035343 | draft | |
| Glamping på TimeWinder 2027 | 2032249 | draft | |
| Test af alt | 2036254 | published, not public | test event; one attendee with entrance/exit/entrance scans on "Scanner 1"; universal exit code 2026-10-23 |
| 2026703 / 2026706 | — | canceled | earlier duplicates of the festival/test event |

2026 events (1897617 festival, 1885612 camping, 1900594 udstiller, 1885132 markedsboder) are `completed`
test shells; 1899540 and 1902059 are canceled.

Account-level objects seen: target groups "Udstillere 2027" (250 unique codes), "Udstillere 2026",
"Super fans", "Fans", "Engangskøbere", "Følgere"; campaigns incl. "Early Bird" (30 % effect, state
prepared, attached to the 2026 markedsboder event), "Udstiller koder 2027", "VW2027 kode", "Handicap
ledsager/billet", several "Access Code TWTW/twtw" duplicates and old "Familierabat" campaigns (the family
campaign was reportedly deleted; the API still lists it — verify in the UI). No webhooks configured.
Festival ticket types seen: "Voksen/Barn, endags billet, fredag." (package items each with its own id),
"Handicap ledsager" (max 2 per order, quantity 10 000).

Family packages exist for Friday ("Familie rabat, fredags billet. 2 voksne & 2 børn", one duplicate 0 kr
test variant to be hidden) and for the festival (2+1, 2+2, 2+3 children at 450 kr before fees in Mikkel's
reading of the setup; 250 kr package mentioned for Friday). Verify prices in the event editor before quoting
them anywhere.

## Decisions already taken

- Keep three separate events and join them with Basket; a "Køb billetter" button on the website links to
  the festival event page (no embedded checkout, because Basket does not work in the widget).
- Account widget in list view, "no direct sales", only as an overview.
- Deposit = separate ticket type; refund list sent to Billetto support after the festival; hold-back to be
  agreed in the Adyen onboarding.
- Scanning: Billetto Access Control on Android (Zebra TC57), terminal restrictions "Fredag" vs
  "Lørdag/søndag", Multi Scanner for camping/exhibitor events.
- Ticket information to guests: PDF banner (480 × 730 px) + organiser text with links in the order mail; TW
  keeps its own PDF splitter.

## Open items (as of the snapshot)

1. Adyen onboarding meeting (≈ 30 min) not held; blocks hold-back agreement and rolling payouts.
2. Hide the 0 kr family test package.
3. Ticket banner graphics and order-mail text: no owner after Lene declined.
4. Widget calendar opens on today's date; Billetto could not confirm the internal setting for the account
   widget. List view chosen instead.
5. API key pair created 2026-10-25 and stored in `tw-opshub-kv` (`BILLETTO-API-KEY-ID`,
   `BILLETTO-API-KEY-SECRET`); no webhook configured yet.
6. Smart Link for the website button not yet created (needed to separate site sales from marketplace sales).
7. All 2027 events are `public: false`: Basket and the public endpoint will not work until they are made
   public. Decide the go-public moment as part of the launch plan.
8. The BI report endpoint returns 404 for TimeWinder's events; ask Billetto whether it is plan-gated.
9. Duplicate "Access Code TWTW/twtw" campaigns and old "Familierabat" campaigns still exist in the API
   listing; clean up so reports stay readable.
11. Scanning hardware (Carsten → Mikkel 2026-10-24/25): "Terminal-enhed" mode in Billetto Access Control
    does not pick up Zebra DataWedge input; Billetto has no documented configuration and escalated to tech
    (model, Android version, app version requested). Unresolved.
12. Counting free/unknown exhibitors at the gate: "Skan stregkode ud" is exit-only (needs a prior entrance
    scan). Billetto suggests one unique free ticket per exhibitor, or the Android app's "allow multiple
    scans of the same ticket" (auto exit + new entrance); whether each repeat scan is stored separately and
    exportable is with Billetto tech. Unresolved.
13. Multi Scanner's 36-hour window vs. camping arrival on Wednesday (two days before the Friday entry
    tickets): camping/nostalgicamping/markedsboder events may need different start dates to be scannable
    together; unclear whether `starts_at` can be edited on a published event (API cannot: PATCH is 404).
14. API access is self-service (Administrer → Integrationer → Developers); a user needs "fuld adgang til
    Zapier, API og webhooks" under team/user permissions to see it. Thomas has it (created the keypair
    2026-10-25). Mikkel's written answer frames the API as fetching events, tickets and deposit tickets
    (read); no written promise of ticket issuance exists in the mail record.
10. Exhibitor approval flow (Power Automate, team leads approve udstillere): the API cannot issue the ticket.
    Redesign as: approval → `POST /organiser/target_group_members` with a fresh unique code in
    "Udstillere 2027" → mail with event link + code → exhibitor redeems the 0 kr Udstillerbillet → confirm
    via `order_lines[].used_codes` or the `order.completed` webhook. Clarify with Mikkel what "bestille
    direkte" referred to (GUI "Generer billetter" is the only direct send, 20 per batch).

## Website status

timewinder.dk runs WordPress 7.1 (theme Inspiro, Kadence/Essential/VK blocks, AIOSEO, Site Kit, WP Super
Cache, All-In-One Security) at webtohosting.dk. A test page `/test-billet-test/` links to the Nostalgi
Camping event. Public pages show the 2027 dates but no prices or buy links yet. The Eventin plugin is
installed with demo content and should be cleaned up or removed once Billetto is the sales channel.
