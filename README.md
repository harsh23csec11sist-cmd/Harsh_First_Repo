# BugPilot — Website QA & Bug Finder

BugPilot is a local, browser-powered website quality scanner. Enter a public website URL, choose a quick or deep scan, and review a saved report of likely problems with suggested next steps.

**This is not a Django application.** The stack is Node.js, Express, React, Vite, Playwright and SQLite.

## What it checks

- **Availability and HTTP:** page navigation failures, 4xx/5xx document responses and failed page resources.
- **Browser/runtime:** uncaught JavaScript exceptions, console errors and failed browser requests.
- **Crawl:** same-origin links are discovered and visited up to the selected page/depth limits. Common destructive paths (such as logout, delete and checkout) are excluded from crawling.
- **Accessibility basics:** images without an `alt` attribute, form controls without a detected label, unnamed buttons/links and missing H1 headings.
- **SEO basics:** missing page titles/descriptions and unusually long titles.
- **Responsive layout:** checks for horizontal overflow at a 390px mobile viewport.
- **Performance signal:** browser load-event timing, flagged when it exceeds five seconds.
- **Security headers:** checks for HSTS on HTTPS pages, Content Security Policy, `X-Content-Type-Options` and `Referrer-Policy`.
- **Workflow:** local scan history, a live progress panel, severity filters, issue search, and CSV/JSON export.

Findings are saved in `server/data/bugpilot.sqlite` and remain available after restarting BugPilot.

## Requirements

- Node.js 22.13 or newer. Node.js 26 LTS is recommended.
- npm (included with Node.js).
- Internet access for installing npm packages and the Playwright Chromium browser.

Playwright browser setup is a one-time step on each machine.

## Run on Windows

1. Install a current Node.js LTS release from <https://nodejs.org/>.
2. Clone or download this repository.
3. Open PowerShell in the project folder.
4. Run:

```powershell
npm install
npm run browser:install
npm run dev
```

5. Open <http://127.0.0.1:5173>.

The dashboard runs through Vite, while the Express API runs at `127.0.0.1:4000`.

### Windows one-click helper

Double-click `install.bat` once to install dependencies and Chromium. After it completes, double-click `start.bat` to launch the development servers.

## Production-style run

```powershell
npm install
npm run browser:install
npm run build
npm start
```

Open <http://127.0.0.1:4000>.

## How to use

1. Enter a full URL, for example `https://example.com`.
2. Choose **Quick** (up to 5 pages) or **Deep** (up to 20 pages).
3. Confirm that you own the website or have permission to test it.
4. Select **Start scan**.
5. Select a scan from Recent scans to open its report.
6. Filter findings by severity, search the findings, and open an issue to read its evidence and suggested fix.
7. Export the report as CSV or JSON.

Only one browser scan runs at a time. Additional scans are queued and processed in order.

## Important limits

BugPilot is an automated QA assistant, not a guarantee that a website is bug-free. It cannot infer every business rule or reliably detect every visual, security, or application-logic defect. It does not submit forms, authenticate to private user accounts, test payment flows, fuzz endpoints, or exploit vulnerabilities. Pages outside the starting origin are not crawled, though browser resources are monitored. The performance timing is a basic browser signal, not a full Lighthouse/Core Web Vitals audit.

Use it only on websites you own or are authorized to test. The scanner blocks localhost, private/reserved IP ranges and non-public hostnames, revalidating browser requests and redirects. It runs locally and binds to `127.0.0.1` by default. Do not expose this development server directly to the public internet.

## Project structure

```text
BugPilot/
├── client/
│   └── src/
│       ├── App.jsx          # Dashboard, scan form, report and issue details
│       ├── main.jsx         # React entry point
│       └── styles.css       # UI styling
├── server/
│   ├── data/                # Created automatically; SQLite database
│   ├── database.js          # Persistence and query helpers
│   ├── index.js             # Express API and single-worker scan queue
│   ├── scanner.js           # Playwright crawler and diagnostic rules
│   └── security.js          # URL validation and private-network blocking
├── index.html
├── package.json
└── vite.config.js
```

## Main API endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/health` | Service status and queue state |
| GET | `/api/stats` | Dashboard counters |
| GET | `/api/scans` | Recent scan history |
| POST | `/api/scans` | Queue a scan |
| GET | `/api/scans/:id` | Report and current progress |
| DELETE | `/api/scans/:id` | Delete a completed/failed scan |
| GET | `/api/scans/:id/export.csv` | Download CSV findings |
| GET | `/api/scans/:id/export.json` | Download full JSON report |

Example scan request:

```json
{
  "url": "https://example.com",
  "mode": "quick",
  "authorized": true
}
```

## Troubleshooting

**Playwright cannot find Chromium**

```powershell
npx playwright install chromium
```

**Port 4000 or 5173 is already in use**

Close the other process or change `PORT` in the environment for the API. Vite's development port can be changed in `vite.config.js`.

**A URL is rejected**

Use a public HTTP/HTTPS site you are authorized to scan. Localhost, private network, reserved IP and internal hostnames are intentionally blocked.

**A scan reports false positives**

Automated browser checks are signals, not proof. For example, a blank page may be intentional; security headers depend on deployment architecture; and unlabeled controls may have accessible names added by custom scripts not detected by a simple DOM audit. Validate each finding manually.
