# Event Calendar

Events Aaron and Stacey are likely to enjoy in Baltimore (and wherever they're traveling). A scheduled Claude task does a full refresh plus the Weekend Events Digest email every Thursday around lunch, and a lighter refresh (no email) every Monday morning.

**Live page:** https://aaronvandorn.github.io/Event-Calendar/

## Using the calendar
- **Month / Week / Day** views. Tap a day in Month to open it. Keyboard: ← → to move, `m` `w` `d` to switch views.
- **Color** = event type (Concert, Film, Museum & art, Theater & dance, Comedy, Festival, Talks & books, Nightlife, Food & markets, Tours & community). Tap a legend chip to hide or show a type.
- **Diagonal stripe** = free.
- **Who's flagging:** the 👤 menu picks Aaron or Stacey on this device (it's remembered). Flags, hiding and the ranking are per person.
- **Flag** each event: ★ Interested puts it on that person's calendar; ✕ Not interested hides it for that person only. "Show events I marked Not interested" in Settings brings them back.
- **Calendars** (the menu next to Area): All events · ★ Aaron's · ★ Stacey's · ♥ Both interested · ★ Either of us. On event cards, ★ = you're interested, ☆ = the other person is, ♥ = you both are; the detail line shows names (and ✕ if the other person passed).
- **Ten per day.** Each day lists its ten best-fit events. A busier day gets an orange count badge and a "Show N more" button.
- Long runs (exhibitions, theater runs) sit in an **Also on view** drawer on each day so they don't crowd out one-night events.
- **Light/Dark** toggle in the header (light Logic-Rhythm colorway by default; dark is the Tape Loop Deck hardware-console look). Your choice is remembered per browser.

## How flags teach the agent
Flags save in your browser instantly, tagged with who set them. To share them with the weekly agent (and your other devices), open **Settings → Sync flags to GitHub** and paste a fine-grained personal access token:
1. Open https://github.com/settings/personal-access-tokens/new in a web browser (the GitHub phone app doesn't offer this). It's in your account settings, not the repo's Settings tab: profile picture → Settings → Developer settings (very bottom of the left sidebar) → Personal access tokens → Fine-grained tokens.
2. Give it a name (e.g. "Event Calendar flags") and an expiration. When it expires, flags stop syncing until you paste in a new one.
3. **Repository access:** choose **Only select repositories**, then pick `Event-Calendar`. (Don't leave it on *Public repositories*: that option is read-only, so "Read and write" never appears.)
4. **Permissions:** set **Contents** to **Read and write**.
   - Older layout: expand *Repository permissions*, find **Contents** and change it from "No access" to **Read and write**.
   - Newer layout: click **+ Add permissions**, tick **Contents**, then change its access from "Read-only" to **Read and write**.
   - **Metadata: Read-only** is added automatically; that's expected. Leave everything else alone.
5. **Generate token** and copy it right away (GitHub shows it only once).
6. On the calendar: ⚙ Settings → *Sync flags to GitHub* → paste → **Save**. When the browser offers to save it as a password, accept.

**Phone and laptop:** the token is stored per browser, but you only paste it once if you let your password manager save it. In Chrome signed into your Google account, the saved token syncs through Google Password Manager and the calendar picks it up automatically on your other devices (on iPhone, tap the password field in Settings and choose it from the autofill bar, then Save). Flags themselves sync through the repo, so both devices see the same Interested / Not interested marks.

Each Thursday the agent runs `scripts/calendar_tool.py learn`, which turns each person's flags into their own category, venue and tag weights, and searches harder for what either of you — and especially both of you — mark Interested. The digest marks picks ★ Aaron, ★ Stacey or ♥ Aaron & Stacey.

**Stacey's setup:** open the calendar on her phone, pick **Stacey** in the 👤 menu, and paste a token in Settings (best: a second token made the same way, so either can be revoked on its own).

## What the agent has learned
Settings → *What the agent has learned* shows a plain-language summary for each person, plus a live line computed from current flags. Edit it and Save (needs the token): the agent treats your edits as fact and rewrites the rest each run.

## Trips
Settings → *Trips & other areas* → place + dates (needs the token), or just tell Claude. The next weekly run researches that city for those dates and adds it to the Area menu.

## Your calendar elsewhere
Rebuilt every week — subscribe in Google or Apple Calendar:
- `https://aaronvandorn.github.io/Event-Calendar/interested-aaron.ics`
- `https://aaronvandorn.github.io/Event-Calendar/interested-stacey.ics`
- `https://aaronvandorn.github.io/Event-Calendar/interested-both.ics`
- `https://aaronvandorn.github.io/Event-Calendar/interested.ics` (either of you)

Settings also downloads a snapshot of the current person's flags.

## Files
- `index.html` — the whole site.
- `data/events.json` — events. `data/areas.json` — home + trips. `data/feedback.json` — flags per person. `data/people.json` — who flags. `data/profile.json` — taste notes + learned weights. `data/digest.json` — email recipients.
- `scripts/calendar_tool.py` — validate, prune, learn, build `interested.ics`, build the digest.
- `digests/` — each week's email (HTML + text).
- `AGENT.md` — the weekly runbook the scheduled task follows.
