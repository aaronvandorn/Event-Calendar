# Event Calendar

Events Aaron is likely to enjoy in Baltimore (and wherever he's traveling), refreshed every Thursday by a scheduled Claude task, with a Weekend Events Digest email.

**Live page:** https://aaronvandorn.github.io/Event-Calendar/

## Using the calendar
- **Month / Week / Day** views. Tap a day in Month to open it. Keyboard: ← → to move, `m` `w` `d` to switch views.
- **Color** = event type (Concert, Film, Museum & art, Theater & dance, Comedy, Festival, Talks & books, Nightlife, Food & markets, Tours & community). Tap a legend chip to hide or show a type.
- **Diagonal stripe** = free.
- **Flag** each event: ★ Interested puts it on *My calendar* (switch with the "All events / My calendar" menu); ✕ Not interested removes it. "Show events I marked Not interested" in Settings brings them back.
- **Ten per day.** Each day lists its ten best-fit events. A busier day gets an orange count badge and a "Show N more" button.
- Long runs (exhibitions, theater runs) sit in an **Also on view** drawer on each day so they don't crowd out one-night events.
- **Light/Dark** toggle in the header (dark hardware-console look by default; light uses the Logic-Rhythm colorway).

## How flags teach the agent
Flags save in your browser instantly. To share them with the weekly agent (and your other devices), open **Settings → Sync flags to GitHub** and paste a fine-grained personal access token:
1. Open https://github.com/settings/personal-access-tokens/new in a web browser (the GitHub phone app doesn't offer this). It's in your account settings, not the repo's Settings tab: profile picture → Settings → Developer settings (very bottom of the left sidebar) → Personal access tokens → Fine-grained tokens.
2. Give it a name (e.g. "Event Calendar flags") and an expiration. When it expires, flags stop syncing until you paste in a new one.
3. **Repository access:** choose **Only select repositories**, then pick `Event-Calendar`. (Don't leave it on *Public repositories*: that option is read-only, so "Read and write" never appears.)
4. **Permissions:** set **Contents** to **Read and write**.
   - Older layout: expand *Repository permissions*, find **Contents** and change it from "No access" to **Read and write**.
   - Newer layout: click **+ Add permissions**, tick **Contents**, then change its access from "Read-only" to **Read and write**.
   - **Metadata: Read-only** is added automatically; that's expected. Leave everything else alone.
5. **Generate token** and copy it right away (GitHub shows it only once).
6. On the calendar: ⚙ Settings → *Sync flags to GitHub* → paste → **Save**.

Each Thursday the agent runs `scripts/calendar_tool.py learn`, which turns your flags into category, venue and tag weights, and searches harder for what you mark Interested.

## Trips
Settings → *Trips & other areas* → place + dates (needs the token), or just tell Claude. The next weekly run researches that city for those dates and adds it to the Area menu.

## Your calendar elsewhere
`interested.ics` is rebuilt every week. Subscribe to `https://aaronvandorn.github.io/Event-Calendar/interested.ics` in Google or Apple Calendar, or download a snapshot from Settings.

## Files
- `index.html` — the whole site.
- `data/events.json` — events. `data/areas.json` — home + trips. `data/feedback.json` — your flags. `data/profile.json` — taste notes + learned weights. `data/digest.json` — email recipients.
- `scripts/calendar_tool.py` — validate, prune, learn, build `interested.ics`, build the digest.
- `digests/` — each week's email (HTML + text).
- `AGENT.md` — the weekly runbook the scheduled task follows.
