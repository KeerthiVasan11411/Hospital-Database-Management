# HealthData Pro — Landing Page + Google Sheets Lead Capture

A responsive marketing landing page, recreated pixel-for-pixel from a Figma design, wired to a serverless lead-capture backend built on Google Apps Script. No server, no database, no hosting cost — form submissions are appended directly to a Google Sheet.

---

## Overview

This project converts a Figma design into a production-ready static landing page and connects its lead form to Google Sheets for zero-infrastructure data collection. It was built as an AI-assisted design-to-code pipeline: the visual design was extracted directly from Figma's node tree, translated into semantic HTML/CSS, and paired with a Google Apps Script Web App that acts as a lightweight API endpoint.

**Stack:**
- Plain HTML5 / CSS3 / vanilla JavaScript (no build step, no framework, no dependencies)
- Google Apps Script (`Code.gs`) as the backend
- Google Sheets as the data store

---

## Architecture

```
┌─────────────────┐        GET request         ┌──────────────────────┐        appendRow()      ┌─────────────────┐
│   index.html     │ ─────────────────────────▶ │  Google Apps Script  │ ───────────────────────▶ │  Google Sheet    │
│ (static, no-cors) │   ?name=&email=&phone=    │   Web App (Code.gs)   │   Timestamp/Name/Email/  │  "Responses" tab │
└─────────────────┘                             └──────────────────────┘   Phone/Reason            └─────────────────┘
```

The frontend never receives a readable response from the backend (`mode: "no-cors"`), so success/failure in the UI is inferred from whether the request completed rather than from parsing a JSON body. This trade-off keeps the whole thing deployable as a static file with zero CORS configuration.

---

## Step-by-Step Tutorial

### 1. Extract the design from Figma
Using the Figma MCP (Model Context Protocol) connector, the design node was queried directly by file key and node ID from the Figma share URL:

```
https://www.figma.com/design/<fileKey>/<fileName>?node-id=<nodeId>
```

This returns structured layout data (positions, colors, typography, spacing, component hierarchy) plus exported image/icon assets — instead of relying on a screenshot or manual re-creation.

### 2. Translate design tokens to CSS
Colors, font sizes, spacing, and border-radii extracted from Figma were mapped to CSS custom properties (`:root` variables) so the page mirrors the design system exactly:
- Primary blue `#0058be`, ink `#0b1c30`, body copy `#45464d`, surfaces `#f8f9ff` / `#e5eeff` / `#d3e4fe`
- Type scale and weights from the `Inter` typeface (Regular/Medium/Semi Bold/Bold)

### 3. Rebuild each section as semantic HTML
The page was reconstructed section-by-section to match the Figma layer structure:
- Hero (headline, CTAs, trust badges, bento-style stat visual)
- Achievement/stats band (4 stat cards)
- Features section (copy + feature list + interactive mockup)
- Lead capture form
- Testimonials

### 4. Scope the form to the required fields
Per project requirements, the form was reduced from its original multi-field design down to three fields — **Name, Email, Phone** — while preserving every other section of the design untouched.

### 5. Build the backend (`Code.gs`)
A Google Apps Script Web App was written to:
- Handle both `GET` and `POST` requests through a single handler
- Read `name`, `email`, `phone` (and an optional `reason`) from the request parameters
- Auto-create a `"Responses"` sheet tab with bold, frozen headers on first run
- Append each submission as a new row with a server-side timestamp
- Return a JSON `{ success: true/false }` payload

### 6. Deploy the Apps Script as a Web App
1. Open a Google Sheet → **Extensions → Apps Script**
2. Paste in `Code.gs`
3. **Deploy → New deployment → Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
4. Copy the generated `/exec` URL

### 7. Wire the frontend to the deployed endpoint
The deployed URL was inserted into `index.html` as `SCRIPT_URL`, and the submit handler builds the request as:

```js
const url = SCRIPT_URL + "?" + new URLSearchParams({ name, email, phone });
fetch(url, { method: "GET", mode: "no-cors" })
  .then(() => showSuccess())
  .catch(() => showError());
```

Using `no-cors` avoids CORS preflight issues entirely (Apps Script Web Apps don't return permissive CORS headers by default), at the cost of not being able to read the actual response — so the UI treats "request didn't throw" as success.

### 8. Test end-to-end
Submit the form → confirm a new row appears in the `Responses` tab of the linked Google Sheet with `Timestamp, Name, Email, Phone`.

---

## Project Structure

```
.
├── index.html    # Full landing page (HTML + CSS + JS, single file, no build step)
├── Code.gs       # Google Apps Script backend — deploy via Extensions → Apps Script
└── README.md
```

---

## Setup / Usage

1. Create a new Google Sheet.
2. Go to **Extensions → Apps Script**, paste in `Code.gs`, and deploy as a Web App (see Step 6 above).
3. Open `index.html`, replace the `SCRIPT_URL` constant with your deployed `/exec` URL.
4. Open `index.html` in a browser (works directly from `file://`, or host it on any static host — GitHub Pages, Netlify, Vercel, S3, etc.).
5. Submissions will appear automatically in the sheet's `Responses` tab.

**Note:** Image/icon assets are currently served from Figma's temporary CDN (`figma.com/api/mcp/asset/...`), which expires ~7 days after export. For long-term hosting, download those assets and reference them locally.

---

## AI Tools Used

| Tool | Role in this project |
|---|---|
| **Claude (Anthropic)** | Primary assistant — planned the architecture, wrote all HTML/CSS/JS and the Apps Script backend, and produced this documentation |
| **Figma MCP connector** | Read live design data (layout, tokens, assets, component tree) directly from the Figma file via `get_design_context`, used as the source of truth for the HTML/CSS conversion |
| **Google Apps Script** | Not AI itself, but the serverless runtime the AI-generated backend code (`Code.gs`) targets |

---

## License

Add a license of your choice (MIT is a common default for small static projects).
