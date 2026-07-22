# Kitchen Hub — Setup Guide

This is a single-file kitchen dashboard: clock, weather, a per-person family calendar, a shared to-do/grocery list, a sports scoreboard, a family whiteboard, an idle photo slideshow, and a nightly sleep screen. It's meant to run full-screen on a wall-mounted tablet or an old computer monitor in the kitchen.

It's one HTML file (`kitchen-hub-shareable.html`) with no server or database — everything runs in the browser. All of the household-specific setup — names, city, sports teams, sign-in credentials — is done through an on-screen **Settings panel** (the ⚙ Settings button, top-right), not by editing code. You still need to create two free developer accounts to get real calendar/to-do data, but you paste the results into form fields instead of touching the file itself.

## What works out of the box, no setup required

The clock, weather, sunrise/sunset, word-of-the-day quote, and idle slideshow (using stock placeholder photos) all work immediately, using free public APIs that don't need any sign-in. So does the whiteboard (it saves notes in the browser, per device). Open the file in a browser and these all run.

## What needs setup

Two things need your own free developer accounts before they'll show real data: your Google Calendar (for the per-person calendar columns) and your Microsoft To Do / OneDrive (for the grocery list, to-do list, and photo slideshow). Until you do this, those panels just show sample placeholder content — nothing breaks, it just won't be *your* data yet.

### Step 1 — Open Settings and fill in the basics

Click **⚙ Settings** in the top-right corner. Everything here saves to that browser automatically when you click "Save & Reload" — nothing to type into code.

- **Household name** — shown as "The ___ House" in the top corner.
- **City** — type your city and click **Find**; it looks up the coordinates for you automatically (used for weather and sunrise/sunset), so you don't need to know your own latitude/longitude.
- **Family Calendar Columns** — one row per person. The "Match" field gets checked against your real Google Calendar names/emails once you connect Google (Step 2), so type whatever name or email address that calendar is known by. Use "+ Add Family Member" for more rows.
- **Sports Scoreboard** — optional. Pick a league from the dropdown, then pick the actual team from the second dropdown — no codes to look up. Leave it empty to hide the panel.
- **Dinner planner URL** — optional. This dashboard was originally built with a private Google Apps Script meal planner embedded here. Leave this blank and the Dinner tab disappears entirely. If you want something similar, you'd need to build or find your own web-based meal planner and paste its URL here.
- **Show Cameras tab** — this tab currently shows two placeholder stock photos, not a real camera feed (see "Known limitations" below). Leave checked as a visual demo, or uncheck to hide it.

Leave the Google/Microsoft fields blank for now — that's Steps 2 and 3.

### Step 2 — Create your own Google sign-in (for the calendar)

The Google/Microsoft sign-in IDs are not shared credentials — each household needs its own, and it's free.

1. Go to [console.cloud.google.com](https://console.cloud.google.com) and create a new project (any name).
2. Go to **APIs & Services → Library**, search for "Google Calendar API", and enable it.
3. Go to **APIs & Services → OAuth consent screen**. Choose "External," fill in the basic required fields, and add your own Google account as a test user (this keeps it free and avoids Google's app review, since it's just for your household).
4. Go to **APIs & Services → Credentials → Create Credentials → OAuth client ID**. Choose "Web application."
5. Under "Authorized JavaScript origins," add the exact URL you'll host this page at (see Step 4) — for example `http://localhost:8000` for local testing, or your real hosting URL.
6. Copy the generated Client ID, go back to **⚙ Settings** in the app, and paste it into the "Google OAuth client ID" field.

### Step 3 — Create your own Microsoft sign-in (for To Do + OneDrive)

1. Go to [entra.microsoft.com](https://entra.microsoft.com) (or [portal.azure.com](https://portal.azure.com) → Microsoft Entra ID) — this works with a normal personal Microsoft/Outlook account, no paid subscription needed.
2. Go to **App registrations → New registration**. Any name is fine. For "Supported account types," choose the option that includes personal Microsoft accounts (so it works for a regular @outlook.com/@hotmail.com/@live.com login).
3. Under **Authentication**, add a "Single-page application" platform and set the Redirect URI to the same URL you'll host this page at.
4. Under **API permissions**, add the delegated permissions `Tasks.Read` and `Files.Read` (Microsoft Graph).
5. Copy the "Application (client) ID" from the app's Overview page, and paste it into the "Microsoft application (client) ID" field in **⚙ Settings**.
6. Leave the "Microsoft tenant ID" field as `common` — that works for both personal and work/school Microsoft accounts. Only change it if you specifically registered the app under one organization's tenant.
7. In Microsoft To Do (the app or web version), make sure you have a list named something like "Groceries" and one named "Home To Do" — the dashboard looks for lists matching those names.
8. Back in Settings, set the "OneDrive photo folder name" to a folder in your OneDrive containing photos for the idle slideshow (default: "Kitchen Hub Photos").

Once both are filled in, click **Save & Reload** at the bottom of Settings — the page reloads and Google/Microsoft sign-in will pick up the new credentials.

### Step 4 — Host the file (this part is required, even for local use)

Google and Microsoft sign-in will **not** work if you just double-click the HTML file and open it as `file://...` in your browser — both providers block sign-in from local files as a security measure. The page needs to be served over `http://` or `https://`, even if that's only on your own network. A few easy options, roughly in order of simplicity:

- **Quickest test (temporary, resets on reboot):** open a terminal in the folder with the file and run `python3 -m http.server 8000`, then visit `http://localhost:8000/kitchen-hub-shareable.html` in a browser on the same computer.
- **Free permanent hosting:** drag the file onto [Netlify Drop](https://app.netlify.com/drop), or push it to a GitHub repository and enable GitHub Pages. Either gives you a permanent `https://` URL — use that URL as the "Authorized origin" / "Redirect URI" in Steps 2 and 3.
- **An always-on home option:** run a tiny web server on a Raspberry Pi or old computer on your home network (`python3 -m http.server` works fine for this too, just leave it running), and point the kitchen tablet at that machine's local IP address.

### Step 5 — Set it up on a tablet or wall display

The dashboard is designed to run full-screen and stay on. Neither iOS nor Android will do this by default, so:

- **iPad/iPhone:** open the page in Safari, tap Share → "Add to Home Screen," then launch it from that icon (it opens without browser chrome). To keep it from locking/sleeping, use Settings → Display & Brightness → Auto-Lock → Never (best on a device that's dedicated to this and always plugged in), or use Guided Access (Settings → Accessibility) to pin the app open.
- **Android tablet:** install a free kiosk browser app such as "Fully Kiosk Browser," point it at your hosted URL, and enable its auto-start-on-boot and screen-always-on options.
- **Old laptop/monitor:** open the page in any browser, press F11 for full screen, and adjust your OS power settings so the screen never sleeps.

## If you ever need to reconfigure

Everything you enter in Settings is stored in that browser only (not in the file, and not synced anywhere). If you switch to hosting the file somewhere new, or open it in a different browser, you'll need to fill in Settings again there. There's a "Reset to Defaults" button in Settings if you want to start over. For anyone comfortable editing code, the same defaults also live in a `DEFAULT_CONFIG` block near the top of the `<script>` section — editing that changes what shows up before Settings has been saved for the first time, but the Settings panel is the supported way to configure this day to day.

## Known limitations

The Cameras tab is currently a visual demo only — it shows two stock photos with a "Demo" badge, not a real video feed. Ring's live-view API isn't something an individual developer can sign up for on their own (it requires a formal partner agreement with Ring), so wiring up real doorbell/floodlight cameras here would mean either pursuing that partnership or using a different camera brand that offers a public snapshot/streaming API. The Dinner tab is only useful if you build or already have your own web-based meal planner to point it at. The family whiteboard saves notes in the browser's local storage on that one device — it does not sync between multiple tablets or phones, and neither do your Settings — each device/browser needs its own Settings filled in once. And each household setting this up needs its own Google and Microsoft sign-in credentials from Steps 2–3 above; they aren't something one person can hand off to another.
