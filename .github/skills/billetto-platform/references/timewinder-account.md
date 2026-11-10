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
| Udstillerbillet 2027 | 2028490 | published, not public | available 2 970; codes via target group "Udstillere 2027". **No longer the exhibitor import target** — moved to 2026708 types 5383883–5383886 (2026-11-06) |
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

### Ticket types in 2026708 *(read via the API 2026-11-10, 42 types)*

| Id | Name | Type | Price | Hidden (`vip`) | Used by |
|---|---|---|---|---|---|
| 5317707 | Voksen, lørdag eller søndag | Pay | 200 | no | **price source** for personaleforening adult (EPIC-108) |
| 5317708 | Barn, lørdag eller søndag | Pay | 100 | no | **price source** for personaleforening child |
| 5317709 / 5317710 | Voksen / Barn, partout | Pay | 350 / 150 | no | |
| 5317713 / 5317714 | Voksen / Barn, fredag / Aften | Pay | 100 / 50 | no | |
| 5383880 | Barn, partout, gratis til og med 6 år. | Free | 0 | no | |
| 5317711 | Handicap ledsager | Free | 0 | no | |
| 5317712, 5317716 | Politiskolen, test · Tre børn for prisen af et | Free | 0 | yes | |
| 5320462 / 5320467 | Familie rabat, endags billet. Lørdag eller søndag.  2 voksne & 2 / 3 børn (note the **double space**) | Bundle | 500 | no | **price source** for personaleforening family ticket |
| 5320482 / 5320473 | Familie rabat, fredags billet. 2 voksne & 2 / 3 børn | Bundle | 250 | no | (5320490 = hidden 0 kr duplicate) |
| 5383883–5383886 | Udstillerbillet - Veteranbil / Klub / LandRace / AeroDrome | Free | 0 | yes | CSV import `vtb/klb/lrc/aer` |
| 5383994 | Sponsor Billet | Free | 0 | yes | CSV import `spn` (quantity 1 200) |
| — | Personaleforening - Voksen lørdag/søndag · Personaleforening - Barn lørdag/søndag | Free | 0 | yes | **not created yet** (2026-11-10) — owner runs `create-ticket-types.browser.js`; CSV import `p???-vok/-brn` |

**Package composition (learned 2026-11-10):** a package's items are ordinary ticket types created right after it,
with generic names and their own prices that sum to the package price. Saturday/Sunday 2+2 (5320462) = "Voksen,
endags billet, weekend." 200 + 200 + "Barn, endags billet, weekend." 100 + a 0 kr `FreeTicketType` "Barn …"; 2+3
(5320467) adds one more 0 kr child. So: the first child is paid, further children are free, and 2+2 and 2+3 cost
the same. Item names repeat across packages — never match scans on the item name alone. The canceled duplicates
2026703 (festival) and 2026706 (Test af alt) still appear in `GET /organiser/events` with 0 types; filter on state.

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
3. Ticket banner graphics and order-mail text: Lene declined; owner decided 2026-11-09 that an agent designs
   them (Canva, like the free tickets 2026-11-07) and the owner only approves test tickets (T-108-003).
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

## Update 2026-11-05 (from tha@timewinder.dk mail + SharePoint, read via Microsoft 365)

**Billetto's written answers (Mikkel Sørensen, 2026-10-27 and 2026-11-05):**

- `/api/organizer/events/{id}/tickets/generate` is an **internal dashboard function** (active dashboard
  session or a Billetto-issued OAuth token only). No key-pair type gives access; a public equivalent would
  be a new product feature with no timeline. Supported ways to issue tickets: "Generér billetter" in the
  dashboard and **CSV import of tickets**.
- **CSV ticket import is enabled on TimeWinder's organiser account** (support, 2026-11-05; Mikkel "åbnet op"
  2026-11-04). Verified by Thomas: the approval flow now creates ticket data, the CSV is imported in
  Billetto, and the imported barcodes scan correctly. The import itself stays a manual step in Billetto.
  Validation rules for external id and barcode in the CSV are still unanswered; so is how transferred or
  refunded tickets (barcode may change) are handled.
- **Own ticket design is allowed** (confirmed): TimeWinder may send its own ticket with its own barcode/QR.
- **Webhook creation with the key pair is fixed in production** (the 500 from 2026-10-25) — retest.
- Still open with Billetto: parent-package relation in API/webhooks; the **504 on
  `/api/organizer/target_groups/search` in the campaign editor** (blocks the campaign for exhibitors/stalls
  that pay after approval); AI chat-agent questions (model, data processing, statistics, export).
- Multi Scanner: **do not recreate events or move dates**; keep real start/end dates. Product team is
  deciding whether the 36 h window can change or events can be grouped another way. Repeated scans of the
  same ticket are stored as separate rows in the scan export (time, terminal, change type, barcode), but the
  export lacks the ticket type and "Total in venue" is unclear — tech case still open before "one shared
  ticket for anonymous exhibitors" can be the final design.
- Scanner hardware in the tech case: Zebra TC57 (Android 10) and TC58 (Android 16), Access Control 1.0.1;
  DataWedge and the universal exit code are being followed up. Scanning/POS to be tested with Bengta.

**Events seen in mail (not in the API table above):**

| Event | Id | Notes |
|---|---|---|
| Stumpemarked på TimeWinder 2027 | 2041198 | created by Carsten 2026-10-28 together with Markedsboder 2035343; both sent to Karen Veith / Hanne Christiansen for review |

**Ticket types and prices 2027** (owner document "Priser billetter og regler 2027.docx", Carsten 2026-10-22;
red = not validated): friday adult 100 / child 50 online (130/80 at the door), family friday 250; weekend
adult 200 (230 door), evening from 17:00 100 door only, partout fri–sun 350; child 100/130, evening 70,
partout 150; family one-day 500 (530 door). Door sales +20 kr/day. Camping with caravan 800, tent 400,
glamping 1 600 (2 persons), nostalgicamping 800/400, Thursday 16:00 to Monday 12:00. Market stalls:
standard pitch 6×4 m no tent 1 800, 36 m² with tent 5 500, with tent+floor 6 600, single day 1 000, tables
200 each (all incl. 2 partout or 2 one-day tickets). Stumpe-/loppemarked: partout pitch 900, one-day 650,
power 200. Exhibitor ticket 300 with 100 % discount for one person per exhibited object; Dutch trucks 890.
Clubs: car clubs partout 300, 100 % max 5 per purchase; Ældre Sagen 10 %; F5 Saturday 160/80; Ford Classic
Car Club 10 %; Swedish groups and Politihistorisk Forening ≈ 33 % on one-day via special link / invoice;
Hundested Bio 20 %; Classic Car House 10 % code CCH2025. Rules: free entry when bringing a period vehicle
("Solskins udstillere" — access only on the day the object is brought, Sat or Sun); handicap companion
accepted; pilots + guest get board and lodging.

**Early Bird 2027 (owner, 2026-10-22):** 20 Nov – 24 Dec 23:59, max 250 total, "spar momsen" (20 %), on
one-day tickets and the family ticket. Committee wants the website sales pages live by 20 Nov.

**Setup runbook (`Billetto billet oprettelse.docx`, SharePoint, updated 2026-10-29)** also documents: the
sekretariat shared login (do not copy), widget colours #056FDA/#FFFFFF, unique codes (Promover → Audiences →
Manuel målgruppe → Generer koder, antal pr. kode = 1 → campaign type Målgruppe, 100 %), family package
creation (items "separat billet sendes = JA", wrong item count ⇒ start over), booking questions (Design &
tilpasninger → Deltager information; per ticket type; ask once per order vs per ticket), vouchers/add-ons
(Rediger → Billetter → Vouchers, "Brug voucher automatisk på"), deposit as separate ticket type, and the
planned **"Solskins udstillere" event** for free day-of exhibitors: own event, scanner settings "Tillad flere
scanninger af den samme billet" + "Automatisk scan ud når der detekteres en ny scan-in", report via
Skanningsstatistik → XLS export. Open questions listed there: hide a specific event id in the widget;
restrict a ticket to one day in the scanner; "kun billetsælger" ticket; replacement for the 0 kr queue
ticket.

**Exhibitor registrations (SharePoint site `UdstillerData-TW-Team`):** Power Automate form → list
"Register as an exhibitor at TimeWinder" (display name "Exhibitor at TimeWinder"); mails go to the team
mailbox UdstillerData-TW-Team@timewinder.dk; photo uploads in the site's `PictureUploads` folder; the 2024
export has **no registration-plate column** (plates only appear in free text and mails). See
`sharepoint-billetto-lookup/references/plate-sources.md`.

**Market stall approval flows (Power Automate, tested 2026-11-04):** "Vi har modtaget din bestilling af din
Markedsbod" (marked@timewinder.dk) → approval → "Du kan nu bestille din Markedsbillet" with a payment link;
stumpemarked variant from boder@timewinder.dk ("Kræmmer teamet"). On 2026-11-04 the payment link still
pointed at **Place2Book**; switching it to the Billetto stall event is blocked by the campaign 504 above.
