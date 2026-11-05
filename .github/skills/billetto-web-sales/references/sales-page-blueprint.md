# Sales page blueprint — timewinder.dk "Billetter" / "Priser og Åbningstider"

Goal: a visitor decides and clicks through to Billetto in under a minute on a phone. The page sells; Billetto
transacts.

## Section order (top to bottom)

1. **Hero with one CTA**: dates (Pinsen 14.–16. maj 2027), place, "Køb billetter" button (Smart Link to the
   festival event page). Repeat the button sticky on mobile.
2. **Price table**: mirror Billetto ticket types one-to-one — same names, buyer-facing prices including fee,
   sale windows. Group as: Partout · Dagbillet (fredag / lørdag-søndag) · Familiepakker · Børn · Handicap
   ledsager. Mark early bird/drop deadlines with a date, not "snart".
3. **Familiepakker explained**: "2 voksne + 1–3 børn, én billet pr. person, alle skal scannes". This is new
   behaviour compared to Place2Book and will be the top support question.
4. **Overnatning cross-sell**: camping and nostalgicamping as two cards; text "kan lægges i samme kurv som
   entrébilletten" and link to the respective event page (Basket joins them). Deposit explained in one line
   if a deposit ticket type exists.
5. **Udstillere / klubber / venner**: one short block pointing to the code-based entry (access code field is
   on the Billetto page). Do not publish codes.
6. **Praktisk**: opening hours, address, parking, "din billet er en PDF; hver person skal vise sin egen
   stregkode", refund policy (link to Billetto terms + TimeWinder's own), exit/re-entry rule.
7. **FAQ** (5–8 items): fee shown at checkout, Apple Pay on billetto.dk, transfer via Billetto Wallet, lost
   ticket (find it in Billetto), children ages, dogs, accessibility.
8. Footer CTA again.

## Copy rules

- Use Billetto's exact ticket type names in bold; everything else in plain Danish.
- Prices as "250 kr." incl. fee, or state once at the top that fees are added at checkout if the organiser
  chose not to include them. Check `fee_included` per ticket type before writing.
- No "sidste chance" without a real date from a campaign or drop.
- Keep legal text out of the page body; link to it.

## Blocks and components

- Kadence Advanced Buttons for CTAs (consistent styling), Flexible Table Block for the price table (it is
  responsive), Custom HTML for any Billetto snippet.
- Use the Operations Hub live-counter endpoint (when it exists) through a tiny inline script that fills a
  `<span data-tw-counter="partout">`; degrade to static text if the fetch fails.

## Measurement

- Smart Link per button (hero, sticky, footer, camping cards).
- GA4 event via Site Kit: `ticket_cta_click` with `placement` parameter.
- Weekly: compare Billetto report (sales per Smart Link) with GA4 click counts to see where the funnel leaks.

## Launch checklist

- [ ] Event public in Billetto, prices and free-ticket fee verified, 0 kr test package hidden.
- [ ] Page reviewed on mobile, consent declined, WP Super Cache purged.
- [ ] Smart Links created and used; UTM preserved.
- [ ] Eventin demo content removed; no stale Place2Book links anywhere on the site (search the media library
      and footer too).
- [ ] Support mailbox knows the FAQ answers before the page goes live.
