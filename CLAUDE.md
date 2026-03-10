# CLAUDE.md — AI Assistant Guide for `dashboards`

This file provides context and conventions for AI assistants (like Claude) working in this repository.

---

## Project Overview

A **multi-page static dashboard portal** for real-time manufacturing/operations monitoring. It consists of a central hub page (`index.html`) linking to seven specialized dashboards. The entire application is built with plain HTML, CSS, and vanilla JavaScript — no build tools, no frameworks, no package manager.

The UI is in **Ukrainian** and data is pulled from **Google Sheets** via **Google Apps Script** using JSONP.

---

## Repository Structure

```
/
├── index.html                  # Hub/landing page — navigation to all dashboards
├── dashboard.html              # Production & downtime dashboard
├── dashboard_employees.html    # Employees dashboard
├── dashboard_defects.html      # Defects/quality dashboard
├── dashboard_safety.html       # Safety dashboard
├── dashboard_power.html        # Teams/power (brigades) dashboard
├── dashboard_announcements.html# Announcements dashboard
└── dashboard_proposals.html    # Proposals dashboard
```

There are no subdirectories, build artifacts, or configuration files outside of `.git/`.

---

## Technology Stack

| Layer | Technology |
|---|---|
| Markup | HTML5 (semantic elements: `<header>`, `<main>`) |
| Styling | CSS3 — Flexbox, Grid, media queries |
| Logic | Vanilla JavaScript (ES6+) |
| Charts | [Chart.js 3.9.1](https://www.chartjs.org/) via CDN |
| Data backend | Google Sheets + Google Apps Script |
| Data transport | JSONP (for cross-origin requests) |
| Images | Google Drive public share links |
| Hosting | Static file server (Git-based deployment) |

**No Node.js, no npm, no TypeScript, no bundler, no linter, no test runner.**

---

## Architecture

### Hub-and-Spoke Pattern

`index.html` is the entry point. It renders a navigation grid of cards, each linking to a dashboard page. Every dashboard is a **standalone SPA** — fully self-contained with its own styles and scripts.

### Data Flow

```
Browser
  └─► Static HTML file
        └─► JSONP request → Google Apps Script URL
              └─► Google Sheets data
                    └─► Parse JSON → Transform → Render charts/tables
```

- Data refreshes automatically every **5 minutes** via `setInterval`.
- On API failure, dashboards fall back to **hardcoded sample data** for offline/dev use.

### JSONP Pattern (used everywhere)

```javascript
const callbackName = 'jsonpCallback_' + Math.random().toString(36).substr(2, 9);
window[callbackName] = function(data) {
    // handle data
    document.head.removeChild(script); // cleanup
    delete window[callbackName];
};
const script = document.createElement('script');
script.src = SCRIPT_URL + '?callback=' + callbackName;
document.head.appendChild(script);
```

Some dashboards use `fetch()` with `try/catch` fallback to sample data instead.

---

## Key Conventions

### JavaScript

- All script is **inline** inside `<script>` tags at the bottom of each HTML file.
- Google Apps Script URLs are stored in a `const SCRIPT_URL = '...'` at the top of the script block.
- Polling intervals are hardcoded: `setInterval(loadData, 5 * 60 * 1000)`.
- Date/time formatting uses `toLocaleString('uk-UA', ...)` or manual DD.MM.YYYY parsing.
- Console logging is used for debugging: `console.log(...)` / `console.error(...)`.
- No modules, no imports — all code in a single global scope per page.

### CSS

- All styles are **inline** inside `<style>` tags in the `<head>`.
- Responsive breakpoints: `640px`, `768px`, `1024px`, `1400px`.
- Color palette:
  - Primary: `#667eea` (purple-blue)
  - Success: `#10b981` (green)
  - Danger: `#ef4444` (red)
  - Warning: `#f59e0b` (amber)
  - Accent: `#8b5cf6` (purple)
  - Background gradient: `linear-gradient(135deg, #667eea 0%, #764ba2 100%)`
- Font stack: `'Segoe UI', Tahoma, Geneva, Verdana, sans-serif`

### HTML

- Language attribute: `<html lang="uk">` — all visible text is in **Ukrainian**.
- Each page has a `<header>` with the dashboard title and navigation buttons back to hub/other dashboards.
- Navigation buttons are `<button onclick="window.location.href='...'">`.

### Data Structures

Dates come from Google Sheets formatted as `DD.MM.YYYY` or `DD.MM.YYYY HH:MM`. Parse manually:

```javascript
const [day, month, year] = dateStr.split('.').map(Number);
const date = new Date(year, month - 1, day);
```

Duration fields are provided in seconds, minutes, and hours (separate fields: `durationSec`, `durationMin`, `durationHour`).

---

## Making Changes

### Adding a New Dashboard

1. Copy an existing dashboard file (e.g., `dashboard_safety.html`) as a starting point.
2. Update `SCRIPT_URL` to the new Google Apps Script endpoint.
3. Adapt the data transformation logic for the new data schema.
4. Add a navigation card in `index.html` pointing to the new file.
5. Add navigation buttons in the new file linking back to `index.html`.

### Updating an Existing Dashboard

- The Google Apps Script URL is near the top of the `<script>` block.
- Sample/fallback data is in a function named `loadSampleData()` or similar.
- Chart configurations are Chart.js config objects — update `labels`, `datasets`, and `options` as needed.
- To change the refresh interval, find `setInterval(loadData, ...)` and update the ms value.

### Changing Styles

- All CSS is in the `<style>` block in `<head>` of each file.
- There is no shared stylesheet — changes must be applied per-file or across all files manually.
- Prefer existing CSS variables/color values for consistency (see palette above).

---

## Data Sources

Each dashboard connects to a separate Google Apps Script deployment. URLs are embedded directly in the JavaScript of each file:

| Dashboard | Notes |
|---|---|
| `dashboard.html` | Production & downtime data |
| `dashboard_employees.html` | Uses `?sheet=Лист2` query param |
| `dashboard_defects.html` | Quality/defect records |
| `dashboard_safety.html` | Safety incident data |
| `dashboard_power.html` | Brigade/team data |
| `dashboard_announcements.html` | Announcements feed |
| `dashboard_proposals.html` | Improvement proposals |

**Do not commit new Google Apps Script URLs without confirming they are public/authorized deployments.**

---

## Testing & Debugging

There is no automated test suite.

- **Manual testing:** Open the HTML file directly in a browser.
- **API failure simulation:** Disconnect from the internet — dashboards will fall back to sample data.
- **Console inspection:** Open browser DevTools → Console for `console.log` / `console.error` output.
- **JSONP debugging:** Check the Network tab for script requests to `script.google.com`.

---

## Git Workflow

- The `master` branch holds production-ready code.
- Direct file edits are committed with descriptive messages like `Update dashboard_defects.html`.
- No CI/CD pipeline. Deployment is implicit on push to the server.
- When working on a task, use the designated `claude/...` branch and push there.

---

## Localization

- All UI text is in **Ukrainian** (`uk`). Do not change the language of existing UI strings.
- Date/time must use `'uk-UA'` locale when using `toLocaleString`.
- New text added to any dashboard should also be in Ukrainian.

---

## Common Pitfalls

1. **CORS errors** — never use plain `fetch()` to Google Apps Script without a CORS-compatible deployment; use JSONP or ensure the Apps Script returns proper headers.
2. **Date parsing** — Google Sheets exports dates as `DD.MM.YYYY`, not ISO format. Do not use `new Date(dateString)` directly.
3. **Shared styles** — there is no shared CSS file. If you update a color or layout in one file, other files are unaffected.
4. **Chart.js version** — pinned to `3.9.1` from CDN. Do not upgrade without testing all chart types used.
5. **Google Drive images** — use `https://drive.google.com/uc?export=view&id=FILE_ID` format for embedded images. The file must be publicly shared.
