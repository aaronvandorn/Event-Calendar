# Weekly run — Event Calendar

This is the runbook for two scheduled runs (America/New_York):
- **Thursday ~11:40 a.m. — full run:** every step below, including the digest email.
- **Monday ~8:30 a.m. — mid-week refresh:** every step except 4 (no digest, no email). Focus research on the next 10 days and newly announced events.
Goal: refresh `data/events.json` with events Aaron is likely to enjoy, learn from his flags, publish the site, and email the Weekend Events Digest.

## 0. Setup
- Work in a clone of `aaronvandorn/Event-Calendar` (push access). Pull first: `git pull --rebase origin main`.
- Today's date comes from the clock tool, in America/New_York.

## 1. Read the state
- `data/profile.json` — Aaron's tastes (`likes`, `lessLikely`), anyone else's under `people`, plus `learned` weights per person.
- `data/people.json` — who flags on the calendar (Aaron, Stacey).
- `data/feedback.json` — flags set on the calendar. Version 2: `items[id].by[person] = {status, ts}` (`interested`, `not_interested`, or empty = cleared), plus a snapshot of the event's category, venue and tags. An old-style item with a top-level `status` is Aaron's; the tool handles both.
- `data/areas.json` — Baltimore (home) plus any trips (`from`/`to` dates).
- `data/taste.json` — a plain-language summary per person of what you've learned about their tastes. It is shown and editable in the calendar's Settings.
- `data/digest.json` — who gets the email. **Only email `recipients`. Never email anyone in `pending`.**
- Run `python3 scripts/calendar_tool.py learn` and read its per-person summary. Search harder for categories, venues and tags either person flags Interested, and especially for things both flagged (♥). Something one person marks Not interested is fine to keep if the other might like it; things both reject get less coverage.

### Taste summaries (`data/taste.json`)
- If a person's entry has `updatedBy` other than `"agent"`, that person (or their partner) edited it: **every statement in it is authoritative.** Keep those statements, follow them when scoring and researching, and never contradict or quietly drop them. Also fold any new likes/dislikes from the edit into `profile.json` (`likes` / `lessLikely` for Aaron, `people.<id>.likes` for others).
- At the end of every run, rewrite each person's `summary` in plain language (short bullet lines, no jargon, no scores): what they clearly like, what they pass on, any shift since last time, and how many flags it's based on. Preserve user-written statements verbatim or near-verbatim, under a line such as "You told me:". Set `updated` to now and `updatedBy` to `"agent"`.

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
  "address": "1711 N Charles St, Baltimore, MD",   // optional; or add the venue to data/venues.json once
  "map": false,                                     // optional; only for citywide / multi-venue events
  "cost": "Free",                                   // human text as listed: "$15–55", "Pay what you can", "See site"
  "free": true,                                     // true | false | null (unknown)
  "url": "https://…",
  "base": 86,                                       // 0–100 fit for Aaron (see below)
  "summary": "One plain sentence on why it's worth it.",
  "tags": ["architecture", "history"],              // lower-case; reuse existing tags so learning carries over
  "source": "Baltimore Magazine"
}
```

`base` is a household score — fit for Aaron and Stacey together (Stacey's tastes come from her flags until `profile.json → people.stacey.likes` has entries). Guidance: 75–90 strong fit (revival film, adventurous jazz, experimental/electronic nights, historic-building tours, maker/DIY electronics, photography shows); 55–74 good; 35–54 maybe; below 35 only if the day is thin. Apply what `learn` showed: nudge whole categories, venues and tags up or down.

- **Map links:** every event gets a Google Maps link to its venue. Use the same `venue` spelling each time, and when you meet a new venue whose street address you found on its site or a listing, add it to `data/venues.json` (`"Venue name": {"address": "…"}`). Without an address the link searches "venue, city", which usually works. Set `"map": false` (on the event or in venues.json) for citywide or multi-venue events. Never guess an address.
- Update an existing event in place (same `id`) if details changed; don't duplicate.
- Leave events anyone flagged `interested` alone even if they are in the past.
- Set the top-level `"updated"` to now (ISO with offset).
- Run `python3 scripts/calendar_tool.py weekly` (validates, prunes events older than 45 days, refreshes `profile.json → learned` per person, writes `interested.ics` (anyone), `interested-<person>.ics` and `interested-both.ics`). Fix any validation errors.

The page shows at most 10 events per day (highest `base` + the viewer's learned score, their Interested flags first) and flags busier days, so it's fine — good, even — to list more than 10 on a busy day.

## 4. Digest email
- Weekend = the Friday–Sunday after today. Build it:
  `python3 scripts/calendar_tool.py digest --from <FRI> --to <SUN> --area baltimore --area-name Baltimore`
  If a trip overlaps that weekend, also build one for the trip area (`--area <id> --area-name "<Name>" --out digests/<FRI>-<id>`) and send it as its own email.
- The digest ranks by both people's learned scores, puts flagged events first, marks them `★ Name` or `♥ Aaron & Stacey`, and drops events someone rejected unless the other person is interested.
- Read `digests/<FRI>.txt` and sanity-check it: dates right, nothing both rejected, links present.
- Send with the connected Gmail (`send_message`) **to each address in `data/digest.json → recipients` only**, every week, even if it's thin:
  - Subject: `Weekend Events Digest · <Fri Mon D> – <Sun Mon D>`
  - HTML body: contents of `digests/<FRI>.html`; plain-text body: contents of `digests/<FRI>.txt`.

## 5. Publish
- `git add -A && git commit -m "Weekly update <date>"`.
- `git pull --rebase origin main` before pushing — the calendar page commits `data/feedback.json` and `data/areas.json` on its own. On a conflict in either of those two files, keep the remote version and re-run `weekly`.
- `git push origin main`. The site is GitHub Pages from `main` at https://aaronvandorn.github.io/Event-Calendar/

## 6. Finish
On the Monday refresh, skip step 4 and say so in the summary.

Reply with a short summary: events added/updated per area, flags per person and how many both liked, what the learning shifted, and confirmation the email went out (to whom).
