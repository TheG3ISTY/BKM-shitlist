# Faction Hit List — Boykisser Meetup & Static Hearts

A single-file, zero-build static web app for managing a Torn City faction **target list ("hit list")**. It is gated to members of two factions, **Boykisser Meetup `[56875]`** and **Static Hearts `[45990]`**, lets you keep notes per target, pulls **battle-stat estimates** from a community stats API, and surfaces the softest targets first.

Each target records **which faction added it** (auto-detected from the caller's key), plus a **Shared** flag for enemies of both factions. A faction filter (All / Boykisser Meetup / Static Hearts / Shared) colours rows by origin — Boykisser Meetup **blue**, Static Hearts **pink**, Shared **purple**. The **War List** carries a sub-tab per faction so both can run a war at once.

Everything lives in **`index.html`** — vanilla HTML/CSS/JS, no frameworks, no npm, no bundler. It runs by double-clicking `index.html` and behaves the same when served from GitHub Pages.

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | The entire app (HTML + CSS + JS inline). |
| `targets.sample.json` | Example of the import/export format. Copy to `targets.json` to have the app offer to auto-load it. |
| `README.md` | This file. |

---

## Quick start

**Locally:** double-click `index.html` (or open it in any modern browser).

**Hosted:** deploy to GitHub Pages (see [Deploy](#deploy-to-github-pages)) and open `https://<you>.github.io/<repo>/`.

On first load you'll see the **access gate**. Paste your Torn API key to enter.

---

## Access gate

The app shows a gate **before** anything else — the list, the add form, and the **Update Stats** button stay hidden until you're verified.

1. Paste **your own** Torn API key.
2. The app calls `GET https://api.torn.com/user/?selections=profile&key=…` (the Torn API allows browser/CORS requests).
3. If `faction.faction_id` is whitelisted (`56875` or `45990`) → you're in. The key + a "verified" flag are saved in `localStorage` so the gate is skipped on reload.
4. Any other faction (or no faction) → *"Access denied: this tool is for members of Boykisser Meetup [56875] or Static Hearts [45990] only."*
5. A bad key returns an `{ "error": … }` object → *"Invalid API key."*

**A Public (limited/minimal) access key is enough.** This only reads *your own* faction to confirm membership — you do **not** need to share a Full-access key. Get or manage keys at **Torn → Settings → API Keys** (`https://www.torn.com/preferences.php#tab=api`).

The key input is masked (with an optional **Show key** toggle). Use **Log out** in the top bar to forget the key + verified flag and return to the gate (your target list and settings are kept).

> ### ⚠️ The gate is client-side only — NOT real access control
> This is a static page. The faction check runs in JavaScript that **anyone can read and bypass** by copying `index.html` and running it locally. It keeps casual / non-faction users out of the *hosted* app, but it is **not** a security boundary.
>
> Also: **anything committed to a public repo is publicly readable** — including a committed `targets.json`. Your hit list would be visible to the whole internet via the repo, regardless of the gate.
>
> Real data privacy would require a **server-side** faction check (validate the key from a backend you control) and keeping the list **off** the public repo (e.g. a private gist/DB behind auth). This tool intentionally trades that away for zero-infrastructure simplicity.

---

## Using the app

- **Add a target** by typing `Name [ID]`, e.g. `Frenemyx [3981563]`. The numeric ID inside `[…]` is the key for everything; input without `[digits]` is rejected inline. The **Profile** and **Attack** links are generated from the ID (never stored).
- **Edit** the Name and the **Why** note inline (click the cell). Editing a name re-derives its ID and links; an edit that loses the `[ID]` is rejected and reverted. **Why** is purely informational — it affects nothing.
- **Delete** with the ✕ on each row.
- **Sort** by clicking the **Stat Estimate** (default — softest first), **Name**, or **Checked When** headers. `N/A` / never-checked rows always sort to the bottom.
- **Filter** by name or note with the Filter box.
- **Human-readable** toggle switches the stat column between the rounded number (default, sortable) and the human string like `2.99b`.
- The whole list is saved in `localStorage`, so it survives reloads.

### Import / Export

- **Export** downloads `targets.json` (the format in `targets.sample.json`). Commit that file next to `index.html` to share a starter list — *but see the privacy warning above before committing real data to a public repo.*
- **Import** loads a `.json` file (accepts either the `{ "version":…, "targets":[…] }` wrapper or a bare `[…]` array).
- **Auto-load:** when the app is **served** (e.g. GitHub Pages) and a `targets.json` sits next to `index.html`, the app offers a one-time banner to load it. (This won't fire from a `file://` open in Chrome, which blocks local `fetch`; use **Import** there instead.)

---

## Stats API integration

The **Update Stats** button is the *only* thing that touches the stats API. It batches **all** target IDs into requests, then per target sets the stat estimate and stamps **Checked When** (format `DD.MM.YYYY HH:mm`, your local time, 24h).

### Settings (⚙)

- **Stats endpoint URL template** — stored in `localStorage`, never hardcoded into behavior. Use `{KEY}` and `{IDS}` placeholders; `{IDS}` becomes a comma-separated ID list.
  Default:
  ```
  https://ffscouter.com/api/v1/get-stats?key={KEY}&targets={IDS}
  ```
- **Stats API key** — defaults to your verified Torn key (community stat services authenticate with the Torn key). Override only if your provider needs a different key.
- **Test endpoint** — does a single diagnostic request (using your current targets, or a fallback ID) and reports OK / the exact error, so you can debug CORS or a wrong URL without changing your data.

The default targets **[FFScouter](https://ffscouter.com/api-docs)**, whose response matches the shape this app parses:

```json
[
  { "player_id": 267456763, "fair_fight": 5.39, "bs_estimate": 2989885521,
    "bs_estimate_human": "2.99b", "bss_public": 123456, "last_updated": 1747333361,
    "source": "premium", "premium_insights_available": true,
    "distribution": { "distribution_human": "STR (60%) SPD (30%)", "stats_percentage": { "strength": 60, "speed": 30 } },
    "spies": [ { "strength": 1000000, "total": 10000000000, "source": "tornstats" } ] },
  { "player_id": 142625381, "fair_fight": null, "bs_estimate": null, "bs_estimate_human": null,
    "bss_public": null, "last_updated": null, "source": "bss", "distribution": null,
    "premium_insights_available": false, "spies": [] }
]
```

- Each object is matched to a target by `player_id === id`.
- The displayed stat is `bs_estimate` **rounded to the nearest 1000** (kept as a number so the column stays sortable). The human toggle shows `bs_estimate_human` instead.
- **Nulls are valid, not errors.** A player can come back with `bs_estimate: null` and `spies: []` — that target shows **N/A** and *still* gets a fresh **Checked When**. The app never throws or blanks the column on nulls.
- HTTP errors and malformed/non-array JSON produce a specific, visible message.

### Rate limit (hard)

The stats API allows **20 requests per minute per IP**, and the app never exceeds it:

- A normal update is **one** batched request (up to **200** IDs/call).
- Larger lists are auto-chunked and spaced **~3.5s** apart (≈17/min), with a rolling 20-per-60s limiter as a hard backstop.
- After each update the **Update Stats** button shows a short **cooldown** so it can't be spammed.

---

## ⚠️ CORS

`api.torn.com` (the access gate) **allows** browser/cross-origin requests, so the gate works from a static page.

The **third-party stats API may not** send CORS headers. If a stats fetch is blocked, the browser throws and the app shows:

> *"Stats API request blocked by CORS (or the network is down). See README → CORS."*

It does **not** fail silently. Fallbacks if you hit this:

1. **Use an endpoint that allows CORS.** FFScouter's `get-stats` is intended for browser use; confirm with **Settings → Test endpoint**.
2. **Route through a proxy / serverless function** you control (Cloudflare Worker, Vercel/Netlify function, etc.) that adds `Access-Control-Allow-Origin`, and point the endpoint template at it. Example template:
   `https://your-worker.example.com/ffscouter?key={KEY}&targets={IDS}`
3. Avoid public "open" CORS proxies for anything involving your API key — they can read it.

---

## 🔐 Security notes

- **No key is ever hardcoded.** Each user supplies their own Torn key via the gate; it's stored **only** in that browser's `localStorage` and sent only to the Torn API and your configured stats endpoint.
- GitHub Pages is **public**: never commit a key, and remember a committed `targets.json` is world-readable.
- "Verified" state is local to the browser. Logging out clears the key + flag.

---

## Deploy to GitHub Pages

**Via the GitHub UI:** push the repo (commands below), then **Settings → Pages → Build and deployment → Source: _Deploy from a branch_ → Branch: `main` / `/ (root)` → Save.** Your site appears at `https://<username>.github.io/<repo>/` within a minute or two.

The exact git commands are in the next section.

---

## License / disclaimer

Personal faction tool. Torn City is © Torn. Battle-stat estimates come from third-party community services; accuracy is their responsibility. Use within Torn's rules.
