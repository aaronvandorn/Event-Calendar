# Weekly run — Event Calendar

This is the runbook the scheduled agent follows every Thursday around lunch (America/New_York).
Goal: refresh `data/events.json` with events Aaron is likely to enjoy, learn from his flags, publish the site, and email the Weekend Events Digest.

## 0. Setup
- Work in a clone of `aaronvandorn/Event-Calendar` (push access). Pull first: `git pull --rebase origin main`.
- Today's date comes from the clock tool, in America/New_York.

## 1. Read the state
- `data/profile.json` — Aaron's tastes (`likes`, `lessLikely`) plus `learned` weights.
- `data/feedback.json` — flags he set on the calendar (`interested` / `not_interested`), with a snapshot of each event's category, venue and tags.
- `data/areas.json` — Baltimore (home) plus any trips (`from`/`to` dates).
- `data/digest.json` — who gets the email. **Only email `recipients`. Never email anyone in `pending`.**
- Run `python3 scripts/calendar_tool.py learn` and read its summary: categories, venues and tags he flags Interested should get more coverage and higher base scores this week; ones he flags Not interested get less.

## 2. Research
Windows:
- **Baltimore**: today through today + 42 days. Fill gaps first (days with few events), then refresh the coming two weeks.
- **Trips**: every area in `areas.json` whose `to` is today or later and whose `from` is within 120 days. Cover its whole date range. If a new trip has no `timezone`, add the IANA zone (e.g. `America/Lima`).

Search widely, then fetch pages for real dates, times and prices (search snippets are too thin):
- Listings: Baltimore Magazine events calendar (`events.baltimoremagazine.com/calendar/day/YYYY/M/D`), Baltimore Beat biweekly events list, Baltimore Fishbowl "Mark your calendars" monthly roundups, Baltimore Banner things-to-do, Baltimore Sun things-to-do (its pages block fetching — use search results), baltimore.org.
- Film: The Charles (`thecharles.com/movies/revival-series/` and the home page's one-night-only list), SNF Parkway / Maryland Film Festival, Creative Alliance film nights, Sweaty Eyeballs.
- Music: Keystone Korner, Ottobar calendar, Metro Baltimore, Current Space (pools.events links), Baltimore Soundstage, Creative Alliance, Motor House, Peabody Heights, An die Musik, Peabody/Hopkins concerts, Songkick Baltimore. Look specifically for electronic, ambient, experimental and DJ nights.
- Museums: BMA exhibition schedule, Walters (`thewalters.org/?ical=1`), Baltimore Museum of Industry (`thebmi.org/events/?ical=1`), Maryland Center for History and Culture, AVAM, Evergreen/Homewood, B&O Railroad Museum, galleries.
- Talks, tours, community: Red Emma's, Greedy Reads, Bird in Hand, Enoch Pratt calendar, Profs & Pints, Stoop Storytelling, Doors Open, Cylburn, Filbert Street Garden, maker fairs.
- Instagram: follow the Instagram links those listings cite; search `site:instagram.com <venue> <month>` for venues that post flyers only.
- Restaurants and food: pop-ups, markets, special dinners from Baltimore Magazine, Banner and Fishbowl food coverage.
- Trips: the city's tourism board, local newspapers and listings sites, Time Out where it exists, Resident Advisor for club nights, major museums and cinemas, plus Instagram as above.

Rules:
- Never invent a date, time or price. If cost is unknown, write `"See site"` and `free: null`.
- Skip civic meetings, support groups, job fairs, kids-only programming, and anything 21+ bar crawls unless it clearly fits his tastes.
- Prefer the venue's own page for `url`.
- Respect site blocks: if a fetch is refused, use another source; never route around a block.

## 3. Write events
Each event in `data/events.json` → `events[]`:

```json
{
  "id": "2026-10-24-doors-open-baltimore",        // YYYY-MM-DD + slug of title, unique
  "area": "baltimore",                              // id from areas.json
  "title": "Doors Open Baltimore",
  "category": "community",                          // music | film | museum | theater | comedy | festival | talk | nightlife | food | community
  "start": "2026-10-24T10:00",                      // local time at the event; date only + "allDay": true if no time
  "end": "2026-10-24T17:00",                        // optional
  "until": "2027-02-21",                            // optional, for multi-day runs (exhibitions, theater runs, festivals)
  "venue": "52 buildings citywide",
  "cost": "Free",                                   // human text as listed: "$15–55", "Pay what you can", "See site"
  "free": true,                                     // true | false | null (unknown)
  "url": "https://…",
  "base": 86,                                       // 0–100 fit for Aaron (see below)
  "summary": "One plain sentence on why it's worth it.",
  "tags": ["architecture", "history"],              // lower-case; reuse existing tags so learning carries over
  "source": "Baltimore Magazine"
}
```

`base` guidance: 75–90 strong fit (revival film, adventurous jazz, experimental/electronic nights, historic-building tours, maker/DIY electronics, photography shows); 55–74 good; 35–54 maybe; below 35 only if the day is thin. Apply what `learn` showed: nudge whole categories, venues and tags up or down.

- Update an existing event in place (same `id`) if details changed; don't duplicate.
- Leave events Aaron flagged `interested` alone even if they are in the past.
- Set the top-level `"updated"` to now (ISO with offset).
- Run `python3 scripts/calendar_tool.py weekly` (validates, prunes events older than 45 days, refreshes `profile.json → learned`, writes `interested.ics`). Fix any validation errors.

The page shows at most 10 events per day (highest `base` + learned score, Interested always first) and flags busier days, so it's fine — good, even — to list more than 10 on a busy day.

## 4. Digest email
- Weekend = the Friday–Sunday after today. Build it:
  `python3 scripts/calendar_tool.py digest --from <FRI> --to <SUN> --area baltimore --area-name Baltimore`
  If a trip overlaps that weekend, also build one for the trip area (`--area <id> --area-name "<Name>" --out digests/<FRI>-<id>`) and send it as its own email.
- Read `digests/<FRI>.txt` and sanity-check it: dates right, nothing he flagged Not interested, links present.
- Send with the connected Gmail (`send_message`) **to each address in `data/digest.json → recipients` only**, every week, even if it's thin:
  - Subject: `Weekend Events Digest · <Fri Mon D> – <Sun Mon D>`
  - HTML body: contents of `digests/<FRI>.html`; plain-text body: contents of `digests/<FRI>.txt`.

## 5. Publish
- `git add -A && git commit -m "Weekly update <date>"`.
- `git pull --rebase origin main` before pushing — the calendar page commits `data/feedback.json` and `data/areas.json` on its own. On a conflict in either of those two files, keep the remote version and re-run `weekly`.
- `git push origin main`. The site is GitHub Pages from `main` at https://aaronvandorn.github.io/Event-Calendar/

## 6. Finish
Reply with a short summary: events added/updated per area, how many he has flagged, what the learning shifted, and confirmation the email went out (to whom).
