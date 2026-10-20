# Content Calendar — CLAUDE.md

Internal social media content calendar for the **Track app**. Single-file tool, no build step.

## Stack

- **Single file:** `index.html` (vanilla JS, ES modules, no framework)
- **Sync:** Firebase Firestore (`content-calendar--track` project) — real-time, debounced 800ms writes
- **Fallback:** `localStorage` key `cc_v3` used as offline cache and instant load on startup
- **Hosting:** GitHub Pages — push to `main` branch at https://github.com/SaNsK11/content-calendar to deploy; live at https://sansk11.github.io/content-calendar/

## Architecture

Everything lives in one `<script type="module">` block inside `index.html`.

| Layer | Detail |
|---|---|
| State | `localData` object in memory; `load()` returns it, `save(data)` writes it |
| Persistence | `save()` updates localStorage immediately + debounces Firestore write |
| Real-time | `onSnapshot(docRef)` listens for remote changes, skips re-render if an input is focused |
| Firestore doc | `calendar/main` → single field `payload` (JSON string of full data) |

## Data shape

```js
{
  platforms: string[10],          // editable platform names
  "2026-10-20": {                  // weekKey = Monday ISO date
    0: {                           // platform index (0-9)
      0: {                         // day index (0=Mon … 6=Sun)
        posts: [
          { status: string, note: string }
        ]
      }
    }
  }
}
```

## Statuses

`none` → `scripting` → `scheduled` → `posted` → `updated` → `needs-edit` → (cycles back)

## Layout

- **Desktop (>640px):** CSS Grid, `172px repeat(7, minmax(0, 1fr))` — MUST use `minmax(0, 1fr)` not `1fr` to prevent columns expanding with content
- **Mobile (≤640px):** Day-tab selector + per-platform cards for selected day; `.calendar` hidden, `.mobile-view` shown

## Deploying changes

```bash
cd /Users/sanskar/content-calendar
git add index.html
git commit -m "your message"
git push origin main
# live in ~60 seconds at https://sansk11.github.io/content-calendar/
```

## Firebase rules

Firestore is in test mode (open read/write). Rules expire after 30 days from project creation — update them in Firebase Console → Firestore → Rules before they lock out:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /calendar/{doc} {
      allow read, write: if true;
    }
  }
}
```
