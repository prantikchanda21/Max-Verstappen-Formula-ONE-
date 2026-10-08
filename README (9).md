# The Verstappen File — Ultimate

An independent, fan-made Formula 1 archive centred on Max Verstappen. It combines a multi-page **React 19 + TypeScript** frontend with a small **Express 5** API, deployed together as **one Vercel project on one domain**.

It includes a cinematic 3D car, an interactive 3D garage, a race centre backed by live provider data (with bundled fallbacks), recorded qualifying telemetry, a tyre-strategy simulator, a searchable career archive, driver comparisons and an optional Groq-powered "race engineer".

> **Independent fan project.** Not affiliated with, endorsed by or sponsored by Formula 1, the FIA, Red Bull Racing or Max Verstappen. All names and references are used for identification only.

---

## Table of contents

1. [Features](#features)
2. [Pages and routes](#pages-and-routes)
3. [Quick start](#quick-start)
4. [Environment variables](#environment-variables)
5. [Architecture](#architecture)
6. [Project structure](#project-structure)
7. [Data sources and provenance](#data-sources-and-provenance)
8. [API reference](#api-reference)
9. [AI race engineer (Groq)](#ai-race-engineer-groq)
10. [Strategy simulator](#strategy-simulator)
11. [Recorded telemetry](#recorded-telemetry)
12. [Photos](#photos)
13. [Deploying to Vercel](#deploying-to-vercel)
14. [Scripts and commands](#scripts-and-commands)
15. [Testing](#testing)
16. [Troubleshooting](#troubleshooting)
17. [Security and privacy](#security-and-privacy)
18. [Known limitations](#known-limitations)
19. [Attribution and licensing](#attribution-and-licensing)
20. [FAQ](#faq)

---

## Features

| Area | What it does |
|---|---|
| **Cinematic home** | Procedural 3D car, five-light start sequence (skippable and replayable), historical milestones, next published race. |
| **Biography, Legacy, Records** | Chronological driver story through 2024, trophy sculpture and season gallery, record collection and win curve. |
| **F1 101** | Beginner-friendly guide: race weekends, points, tyres, technology, flags, Red Bull history, eras and a glossary. |
| **Race Center** | Season calendar, sessions, race / qualifying / sprint classifications, driver and constructor standings, next-race countdown. |
| **3D Garage** | Four conceptual cars (RB16B, RB18, RB19, RB20 interpretations): orbit, spin, exploded view, hotspots, wireframe, illustrative airflow, camera presets, comparison, fullscreen and PNG capture. |
| **Telemetry lab** | Two recorded Bahrain 2023 qualifying laps (VER vs LEC): replay, scrubbing, channel selection, sector times and interpolated delta. |
| **Circuits** | Artistic notebooks for Austria, Miami and Monza with animated markers. |
| **Strategy simulator** | One/two-stop model with weather, fuel, traffic, safety car / VSC, compound and seeded uncertainty. |
| **Career archive** | Searchable race-by-race archive with full selected-race classification. |
| **Rivals / Comparisons** | Season driver-vs-driver stats, points progression, qualifying head-to-head. |
| **AI engineer** | Groq answers grounded in server-retrieved context, with a deterministic fallback and conversation export. |
| **Accessibility and settings** | Skip link, reduced-motion support (respects device preference), motion toggle, low/high 3D quality, text fallback when WebGL is unavailable. |

---

## Pages and routes

| URL | Experience |
|---|---|
| `/` | Overview, 3D car, starting lights, milestones, next race |
| `/biography` | Interactive chronological driver story |
| `/legacy` | Trophy sculpture and championship season gallery |
| `/records` | Historical records and win curve |
| `/f1-101` | Beginner guide to Formula 1 |
| `/race-center` | Calendar, sessions, classifications, standings |
| `/garage` | 3D car garage |
| `/telemetry` | Recorded Bahrain 2023 qualifying laps |
| `/circuits` | Austria, Miami, Monza notebooks |
| `/strategy` | Tyre and race strategy simulator |
| `/career` | Searchable race archive |
| `/comparisons` | Season driver comparisons |
| `/ai-engineer` | Groq-powered race engineer |

Legacy `.html` URLs redirect to their replacements: `/index.html` → `/`, `/archive.html` → `/career`, `/tracks.html` → `/circuits`, `/engineer.html` → `/ai-engineer`, and `/garage.html`, `/legacy.html`, `/strategy.html`, `/telemetry.html`, `/race-center.html` map to their matching routes. Unknown routes show a "not found" view inside the app.

---

## Quick start

### Requirements

- **Node.js 22 or newer** (enforced by `engines` in `package.json`)
- npm (bundled with Node)
- Internet access for `npm ci` and for live Jolpica data. The app still works offline using bundled snapshots.

No Docker, database, Python or Vercel account is needed for ordinary local use.

### Windows

1. Install Node.js 22+.
2. Extract the project into a normal folder.
3. Double-click **`start.bat`**. It checks Node/npm, runs `npm ci` if needed, copies `.env.example` to `.env`, runs the TypeScript check, starts the API and frontend together, and opens `http://localhost:5173` when `/api/health` responds.
4. Keep the terminal open. Press **Ctrl+C** to stop.

### macOS / Linux / any terminal

```sh
npm ci
cp .env.example .env     # PowerShell: copy .env.example .env
npm run dev
```

Open <http://localhost:5173>.

`npm run dev` runs `scripts/dev.ts`, which starts the **same exported Express app used in production** on `127.0.0.1:5173` and mounts Vite's dev middleware behind it. The frontend and `/api/*` therefore share one origin, just like on Vercel.

### Verify it works

```sh
curl http://localhost:5173/api/health
# {"status":"ok","aiConfigured":false,"cache":"ephemeral-instance + CDN","snapshot":true}

curl "http://localhost:5173/api/results?season=2024&round=1"
```

> **Important:** do not start the frontend with plain `vite` or `npx vite`. That skips Express, so every `/api/*` request returns the HTML app shell and pages show *Data unavailable* with `Unexpected token '<'`. Always use `npm run dev` or `start.bat`.

---

## Environment variables

Copy `.env.example` to `.env` for local use. On Vercel, add them under **Project → Settings → Environment Variables**. Environment variable changes only apply to **new deployments**, so redeploy after editing.

| Variable | Default | Purpose |
|---|---|---|
| `GROQ_API_KEY` | *(empty)* | Optional **secret**. Enables Groq answers on `/ai-engineer`. Server-side only. Never prefix with `VITE_`. |
| `GROQ_MODEL` | `llama-3.3-70b-versatile` in code | Groq model ID. **That default is deprecated** (see [AI race engineer](#ai-race-engineer-groq)). Set `openai/gpt-oss-120b`, or any model your Groq account supports. Not a secret. |
| `JOLPICA_BASE_URL` | `https://api.jolpi.ca/ergast/f1` | Ergast-compatible provider base URL. No trailing path. |
| `DATA_FALLBACK_ENABLED` | `true` | Allow same-season bundled snapshots when the provider fails. |
| `CACHE_TTL_SECONDS` | `3600` | Warm in-memory cache lifetime per function instance. Bounded to 30–86400. Use `86400` to minimise upstream calls. |
| `DATA_OFFLINE` | `false` | Fault injection: `true` skips Jolpica entirely and serves snapshots. Playwright sets this. **Do not enable in production.** |

Only `GROQ_API_KEY` is sensitive. The model name is a public identifier.

---

## Architecture

```
Browser (React 19, Vite build)
   │  same origin
   ├── /                → dist/index.html (React Router handles routes)
   ├── /assets, /datasets, /images → static files
   └── /api/*  ───────► Express 5 app (backend/src/app.ts)
                           │
                           ├── validation (Zod), Helmet, rate limits
                           ├── /api/strategy → shared/strategy.ts (deterministic simulator)
                           ├── /api/ai/chat  → Groq (optional) or deterministic fallback
                           └── racing(season, kind)
                                  1. warm in-memory cache
                                  2. Jolpica with bounded pagination
                                  3. stale warm response from this instance
                                  4. bundled same-season snapshot
                                  5. explicit JSON 503
```

### Request flow for racing data

Implemented in `backend/src/providers/racing.ts`:

1. **Warm cache** — if a fresh entry exists for `season/kind`, return it.
2. **Deduplication** — concurrent requests for the same table share one in-flight promise.
3. **Jolpica** — fetch `…/{season}/{endpoint}.json?limit=100` with a **9.5 s total budget**, **3.5 s per request**, one retry on 5xx. HTTP 429 fails immediately. Further pages are fetched at most two at a time.
4. **Stale cache** — if Jolpica fails and an older response exists in this instance, return it with `source: "stale-cache"` and a warning.
5. **Bundled snapshot** — if `DATA_FALLBACK_ENABLED` is not `false` and a snapshot for exactly that season and kind exists, return it with `source: "bundled"`.
6. **503** — otherwise return JSON: *"No verified data is available for this season. Try 2021–2024 or retry later."*

The app never silently substitutes a different season's data.

### Pagination

Jolpica caps pages at 100 rows even when a larger `limit` is requested. `pagination.ts` fetches all pages (bounded at 1,200 total rows), rejects the response if the total changes mid-way, and **joins split races by round** so a Grand Prix whose results span two pages is merged into one race before statistics are computed.

### Caching layers

- In-memory per warm function instance (`CACHE_TTL_SECONDS`).
- CDN headers: live provider responses use `public, max-age=60, s-maxage=900, stale-while-revalidate=3600`; fallback responses use `max-age=30, s-maxage=60`.
- TanStack Query on the client (15-minute `staleTime`, one retry).

Cache and rate-limit state are **ephemeral across Vercel instances**. No database is needed.

---

## Project structure

```
max-verstappen-ultimate/
├── api/
│   └── index.ts                 Vercel Node function; re-exports the Express app
├── backend/src/
│   ├── app.ts                   Routes, validation, headers, rate limits, Groq grounding
│   └── providers/
│       ├── racing.ts            Cache → Jolpica → stale → snapshot → 503
│       └── pagination.ts        Multi-page fetch, merge by round, dedupe
├── shared/                      Used by both frontend and backend
│   ├── data.ts                  Zod provider schemas, types, query schema
│   ├── editorial.ts             Biography, records, seasons
│   ├── f1.ts                    F1 101 content
│   └── strategy.ts              Deterministic strategy simulator and input schema
├── frontend/
│   ├── index.html
│   ├── public/
│   │   ├── datasets/            Telemetry manifest + two recorded laps (static JSON)
│   │   ├── images/              Background and photo manifest
│   │   └── models/README.txt
│   └── src/
│       ├── App.tsx              Routes, navigation, GSAP page transitions
│       ├── main.tsx
│       ├── pages/               Home, Editorial, Sport, RaceCenter, Garage, Telemetry,
│       │                        Circuits, Strategy, Archive, Engineer
│       ├── scenes/Studio.tsx    Procedural 3D car and trophy (React Three Fiber)
│       ├── components/          UI, Chart (ECharts), Results, Photo
│       ├── services/api.ts      Same-origin API client + TanStack Query hook
│       ├── stores/settings.ts   Motion and 3D quality (Zustand)
│       └── styles/index.css     Tailwind v4 + design system
├── datasets/historical/
│   └── snapshots.json           Bundled provider snapshots (≈4.8 MB)
├── scripts/
│   ├── dev.ts                   Express + Vite middleware dev server
│   ├── preview.ts               Serves dist/ + API on :5173
│   ├── ingest.ts                Refresh snapshots from Jolpica
│   ├── ingest_telemetry.py      Optional FastF1 telemetry importer
│   └── fetch-photos.mjs         Optional Wikimedia Commons photo downloader
├── tests/
│   ├── core.test.ts             Vitest + Supertest API/data/strategy tests
│   ├── deployment.test.ts       Production-shell / routing-order test
│   └── e2e/site.spec.ts         Playwright desktop + mobile scenarios
├── docs/                        DATA.md, TESTING.md, MIGRATION.md
├── vercel.json                  Rewrites, headers, function settings
├── vite.config.ts               Vite root = frontend/, output = dist/
├── playwright.config.ts
├── vitest.config.ts
├── eslint.config.js
├── tsconfig.json
├── start.bat                    Windows one-click launcher
├── .env.example
└── package.json
```

### Technology stack

| Layer | Technology |
|---|---|
| Frontend | React 19, React Router 7, TypeScript (strict), Vite 6, Tailwind CSS 4 |
| 3D | Three.js, React Three Fiber, Drei |
| Motion | GSAP + ScrollTrigger, Motion |
| Charts | ECharts |
| State / data | TanStack Query, Zustand |
| Validation | Zod |
| Backend | Node.js 22, Express 5, Helmet, express-rate-limit |
| Testing | Vitest, Supertest, Playwright |
| Fonts | Barlow / Barlow Condensed via Fontsource (self-hosted, no CDN) |

Dependencies are locked in `package-lock.json`. Always install with `npm ci`.

---

## Data sources and provenance

### Race data (Jolpica)

Provider: <https://api.jolpi.ca/ergast/f1/> (community-maintained, Ergast-compatible). Docs: <https://api.jolpi.ca/docs/>. Source project: <https://github.com/jolpica/jolpica-f1>.

Jolpica is **not an official live timing service**. Results can be delayed or corrected. Displayed timestamps record when data was *retrieved*, not when the source last changed it.

**Rate limits.** Third-party client libraries document the unauthenticated limits as roughly **4 requests/second burst and 500 requests/hour sustained** (verify on Jolpica's docs). Exceeding them returns HTTP 429, which this app treats as a failure and answers from fallback. One season table is ~5 requests, so heavy traffic on cold instances can exhaust the budget. Mitigations: set `CACHE_TTL_SECONDS=86400`, rely on CDN caching, and refresh snapshots after each race.

### Bundled snapshots

`datasets/historical/snapshots.json` stores each table with its source URL, retrieval timestamp and original provider payload. Paged tables are merged by round; `MRData.limit` is set to the merged total.

| Table | Coverage |
|---|---|
| `results`, `standings` | 2015–2026 |
| `calendar`, `constructors`, `qualifying`, `sprint` | 2021–2026 |

Notes:

- Seasons **before 2021 have no bundled calendar, constructors, qualifying or sprint snapshot**. They work only while Jolpica is reachable. The season dropdown lists 2015 onward, so choosing those years with the provider down yields a 503.
- The **2026 snapshot is a partial season**: it contains only what existed at retrieval time.
- An empty sprint table means no published sprint records, not an error.
- The snapshot is a deployment artifact. It does not update until you re-ingest and redeploy.

### Refreshing snapshots

```sh
npm run data:update -- 2026   # fetches all six tables for the season, ~1.1 s between calls
npm test
npm run build
# commit and redeploy
```

The script writes the file only after all tables are collected, so a failed run keeps the previous snapshot. Production functions never modify snapshots.

### Statistical definitions

- **Archive points** sum Grand Prix results only. They exclude sprint points and championship adjustments. The standings endpoint is authoritative for championship totals.
- **Entries** count provider result rows, not guaranteed starts.
- **Head-to-head** compares classified positions, includes retirements and excludes races where both drivers did not appear.
- **Win and podium rates** use recorded entries as the denominator.
- No claim is made that the numbers isolate driver skill from machinery.

### Editorial content

Biography and records are intentionally scoped through **2024** and are not live career totals. Sources include Formula 1's Max Verstappen Hall of Fame profile and the 2024 title report (links in `docs/DATA.md`).

---

## API reference

All endpoints return JSON, including errors. Unknown `/api/*` paths return **JSON 404**, never HTML. Request bodies are limited to **24 KiB**.

| Method | Path | Query / body |
|---|---|---|
| GET | `/api/health` | — |
| GET | `/api/calendar` | `season`, optional `round` |
| GET | `/api/results` | `season`, optional `round` |
| GET | `/api/standings` | `season` |
| GET | `/api/constructors` | `season` |
| GET | `/api/qualifying` | `season`, optional `round` |
| GET | `/api/sprint` | `season`, optional `round` |
| GET | `/api/telemetry` | — (returns the static manifest location) |
| POST | `/api/strategy` | Strategy input (see below) |
| POST | `/api/ai/chat` | `message`, `season`, optional `round`, `history` |

`season` is an integer from 1950 to next year (default 2024). `round` is an integer 1–40.

### Racing response envelope

```json
{
  "season": 2024,
  "kind": "results",
  "races": [ { "round": "1", "raceName": "Bahrain Grand Prix", "Results": [ … ] } ],
  "standings": [],
  "source": "jolpica",
  "updatedAt": "2026-10-08T13:14:44.000Z",
  "sourceUrl": "https://api.jolpi.ca/ergast/f1/2024/results.json?limit=100",
  "partial": false,
  "warning": "optional message when serving fallback data"
}
```

`source` is one of `jolpica`, `stale-cache` or `bundled`. `partial` is `true` if the provider reports more rows than were retrieved.

### Errors

```json
{ "error": "Invalid input", "issues": ["…"] }                       // 400, Zod validation
{ "error": "No verified data is available for this season. …" }     // 503
{ "error": "Unknown API endpoint" }                                 // 404
{ "error": "Request could not be completed" }                       // 500
```

### Rate limits (this app)

- General `/api`: **90 requests/minute**
- `/api/ai/chat`: **10 requests/minute**

These are **per warm instance**. Vercel Firewall rules can enforce broader quotas. No distributed-limit guarantee is claimed.

### Examples

```sh
curl "http://localhost:5173/api/calendar?season=2024"
curl "http://localhost:5173/api/standings?season=2024"
curl "http://localhost:5173/api/qualifying?season=2024&round=10"

curl -X POST http://localhost:5173/api/ai/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"How did Max do in Bahrain?","season":2024,"round":1}'
```

---

## AI race engineer (Groq)

The AI page calls `POST /api/ai/chat`, which runs only on the server.

**How it answers**

1. Retrieves race results for the selected season (and round, if given) through the same `racing()` pipeline.
2. Selects matching biography entries by keyword.
3. Builds a context object (season, race results, championship seasons, biography matches).
4. If `GROQ_API_KEY` is set, sends the context plus up to 10 history messages to Groq's OpenAI-compatible endpoint (`https://api.groq.com/openai/v1/chat/completions`, 15 s timeout, `temperature 0.15`, `max_tokens 850`).
5. The system prompt tells the model to use **only** the supplied verified context for factual racing claims, to say "unavailable" when data is absent, and never to invent telemetry, current totals or citations.
6. If there is no key, Groq errors, times out or returns an unexpected shape, the server returns a **deterministic summary** labelled *"Verified data summary — not an AI-generated answer"*.

The response contains `mode` (`groq` or `deterministic`), `answer` and `sources`.

**Request body**

| Field | Rules |
|---|---|
| `message` | string, 1–1500 characters |
| `season` | integer 1950–next year, default 2024 |
| `round` | optional integer 1–40 |
| `history` | up to 10 items, each `{role: "user" | "assistant", content ≤ 3000 chars}` |

### Choosing the model

The code default (`llama-3.3-70b-versatile`) is **out of date**. Groq announced deprecation of `llama-3.3-70b-versatile` and `llama-3.1-8b-instant` on June 17, 2026, with shutdown reported for August 16, 2026, and recommended `openai/gpt-oss-120b` or `qwen/qwen3.6-27b` as replacements. Model lists change often, so confirm the ID in your Groq console.

A retired model makes Groq return an error, and the app **silently falls back** to the deterministic summary, so a valid key can look like it "isn't working".

To change it, set the environment variable (no code edit required):

```
GROQ_MODEL=openai/gpt-oss-120b
```

On Vercel: Settings → Environment Variables → add/edit `GROQ_MODEL` → **Redeploy**.

Optional code tweaks:

- Change the fallback string in `backend/src/app.ts` and `.env.example` to your preferred model.
- GPT-OSS models are reasoning models; their reasoning can consume the `max_tokens: 850` limit and produce short or empty answers. Raise it (for example to `1500`) if that happens.

### Privacy

When Groq is configured, the conversation and selected racing context are sent to Groq. The app stores no conversations on the server. Responses are not streamed.

---

## Strategy simulator

`shared/strategy.ts` contains a deterministic model that runs **in the browser** and is also exposed at `POST /api/strategy`.

**Model inputs**

| Field | Range / values |
|---|---|
| `laps` | integer 10–100 |
| `base` | base lap time in seconds, 40–200 |
| `pit`, `second` | pit laps (integers); `pit ≥ 2`, `second ≥ 3`, `pit < second < laps` |
| `pitLoss` | seconds, 0–60 |
| `degradation` | 0–1 |
| `compound` | `soft` / `medium` / `hard` |
| `temperature` | °C, 10–65 |
| `weather` | `dry` / `damp` / `wet` |
| `neutralization` | `none` / `sc` / `vsc` |
| `neutralLap` | integer ≥ 1, must be ≤ `laps` |
| `traffic` | 0–3 |
| `fuel` | seconds per lap per unit, 0–0.1 |
| `pace` | offset, −3 to 3 |
| `seed` | integer 1–100000 (seeds the Monte Carlo generator) |

**Defaults:** 57 laps, 92 s base, pit laps 20/38, 22 s pit loss, medium compound, 35 °C, dry, no neutralisation.

**What it models:** compound offsets, tyre-age degradation, linear fuel burn effect, temperature penalty, weather pace offsets, traffic, pit loss, and a bounded SC/VSC window (about 3 and 2 laps). Monte Carlo varies a race-wide pace offset with a seeded generator.

**What it does not model:** wet-tyre selection, race legality rules, overtaking and finishing positions. The uncertainty interval reflects the model's assumptions and is **not calibrated** predictive confidence.

Invalid input (for example `second ≤ pit`) returns HTTP 400 with the validation message.

---

## Recorded telemetry

`/telemetry` plays back two **fastest qualifying laps from Bahrain 2023**:

| Driver | Lap | Lap time |
|---|---|---|
| VER | 14 | 1:29.708 |
| LEC | 16 | 1:30.000 |

Data is prepared with FastF1 from F1 timing data and served as small static JSON in `frontend/public/datasets/` (no function call). The manifest lists lap, sector times, channels and source.

- **Channels:** speed (km/h), throttle (%), brake (boolean), RPM, gear.
- **Time** is relative to the lap's car-data slice; endpoint samples may extend slightly past the timed lap.
- **Distance** is integrated from speed by FastF1. It is **not GPS**.
- **Rival delta** uses linear time interpolation at equal integrated distance, so it is approximate.
- **Playback** advances one sample every 200 ms. It is sample playback, not an exact wall-clock reconstruction.
- Track position, steering angle, high-frequency GPS, live timing and tyre temperatures are unavailable and **not synthesised**.

To regenerate or extend telemetry (optional, not needed at runtime):

```sh
python -m pip install fastf1
python scripts/ingest_telemetry.py
```

Do not commit FastF1's cache directory (already in `.gitignore`).

---

## Photos

A background image and a few photos are bundled under `frontend/public/images/`. `frontend/public/images/max/manifest.json` maps photos to page slots.

**Download more (needs internet):**

```sh
npm run photos:fetch           # freely licensed photos from Wikimedia Commons
npm run photos:fetch -- 40     # choose how many
```

**Matching:** each slot first uses a photo whose `slots` list names it (for example `"slots":["home-1"]`), then the photo whose title best matches the slot's keywords (year, circuit, car), then any remaining photo. Slots with no photo show a styled "33" placeholder.

**Your own photos:** copy them into `frontend/public/images/max/` and add entries:

```json
{ "photos": [ { "file": "my-photo.jpg", "title": "Max Verstappen 2023 Monza",
                "credit": "Photographer name", "license": "Licence",
                "slots": ["home-1"] } ] }
```

Only publish images you have the right to use. Creative Commons licences require attribution; keep the credits. Official F1 and agency photos are copyrighted.

---

## Deploying to Vercel

The repository is designed as **one Vercel project**: static frontend plus one Node function.

1. Push the **contents of the project folder** to a GitHub repository (not the parent folder). Do not commit `.env` or `node_modules`.
2. Import the repo in Vercel and select the **repository root**.
3. Settings:
   - Framework preset: **Vite**
   - Install command: `npm ci`
   - Build command: `npm run build`
   - Output directory: `dist`
   - Node.js version: **22.x** or newer
4. Add environment variables (`GROQ_API_KEY`, `GROQ_MODEL`, and optionally `CACHE_TTL_SECONDS=86400`).
5. Deploy, then verify:
   - `/api/health` returns `{"status":"ok",…}`
   - `/api/calendar?season=2024` and `/api/results?season=2024&round=1` return JSON with `source` of `jolpica` or `bundled`
   - `/api/not-a-route` returns a **JSON 404**, not HTML
   - Every page loads; refresh `/garage` directly to confirm SPA routing
   - The AI page returns a Groq answer (after setting a valid key and model)

### How `vercel.json` routes requests

- `/api/:path*` → rewritten to the function `api/index.ts` (`maxDuration: 30`).
- Any other path not starting with `api/`, `assets/`, `datasets/`, `images/` or `favicon.svg` → `index.html`, so React Router can handle deep links.
- `/assets/*` gets `Cache-Control: public, max-age=31536000, immutable`.
- Global headers: `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `X-Frame-Options: DENY`.

### ESM rules for the server code (important)

`package.json` has `"type": "module"`, so the Vercel function runs as native ES modules. Two rules apply to code under `api/`, `backend/` and `shared/` that the function imports:

1. **Relative imports must include the `.js` extension**, even though the source files are `.ts`:
   ```ts
   import {racing} from './providers/racing.js';          // correct
   import {racing} from './providers/racing';             // crashes on Vercel
   ```
   Without it the function fails with `ERR_MODULE_NOT_FOUND` and every `/api/*` call returns `500 FUNCTION_INVOCATION_FAILED`. TypeScript (`moduleResolution: Bundler`), Vite, tsx and Vitest all resolve `.js` to the `.ts` file, so local development keeps working.
2. **JSON imports need an import attribute** in Node 22:
   ```ts
   import snapshots from '../../../datasets/historical/snapshots.json' with {type:'json'};
   ```

Frontend-only files (under `frontend/src`) are bundled by Vite and do not need extensions.

### Local Vercel emulation (optional)

```sh
npx vercel login
npx vercel link
npm run dev:vercel
```

Required only if you want to test Vercel's routing with the CLI. Do not use the Express-only framework preset.

---

## Scripts and commands

| Command | What it does |
|---|---|
| `npm run dev` | Express API + Vite dev middleware on <http://localhost:5173> |
| `npm run preview` | Serves the production `dist/` plus the API on port 5173 (run `npm run build` first) |
| `npm run build` | `tsc --noEmit` then `vite build` into `dist/` |
| `npm run typecheck` | Strict TypeScript check |
| `npm run lint` | ESLint |
| `npm test` | Vitest + Supertest |
| `npm run test:e2e` | Playwright (desktop + mobile) |
| `npm run data:update -- <season>` | Refresh bundled race snapshots for a season |
| `npm run photos:fetch [-- N]` | Download Wikimedia Commons photos |
| `npm run dev:vercel` | `vercel dev` (needs login and link) |

---

## Testing

```sh
npm ci
npm run typecheck
npm run lint
npm test
npm run build
npx playwright install chromium
npm run test:e2e
```

**Unit / API tests (`tests/core.test.ts`)** cover: JSON health and 404 behaviour, invalid seasons and chat input, same-season fallback, unavailable seasons, deterministic AI responses and Groq failure, provider schemas, complete 2024 results, pagination across a split race, concurrent request deduplication, deterministic strategy results, pit-loss arithmetic and SC pit-loss adjustments.

**Deployment test (`tests/deployment.test.ts`)** checks the production shell: deep links, API 404 and emitted assets.

**Browser tests (`tests/e2e/site.spec.ts`)** run six scenarios on desktop and mobile: every route renders without horizontal overflow, race fallback and classifications, strategy inputs and validation, telemetry scrubbing, garage control state, deterministic AI response. Playwright starts the dev server with `DATA_OFFLINE=true`.

See `docs/TESTING.md` for what was and was not verified. Items that still need a real environment: Playwright browser runs, visual and GPU behaviour, live Groq calls, and an actual Vercel deployment.

Manual checks worth doing in a real browser: high and low 3D quality, reduced motion, PNG capture, fullscreen, touch orbit, chart zoom and direct route refresh.

---

## Troubleshooting

### "Data unavailable" on Race Center, Career, Comparisons

Those pages load data through `/api/*`, while Telemetry reads static files, so Telemetry can work while the others fail. The red box prints the real error underneath:

| Message | Meaning | Fix |
|---|---|---|
| `Unexpected token '<'…` or `Unexpected end of JSON input` (older builds) / `API not reachable (HTTP …)` | `/api` returned HTML or crashed instead of JSON | Run `npm run dev` / `start.bat` locally, or fix the function on Vercel (below) |
| `No verified data is available for this season…` | Backend works, Jolpica failed, and no snapshot exists for that season and table | Pick 2021 or later, or run `npm run data:update -- <year>` |
| Network or timeout error | Browser cannot reach the site | Check the connection or deployment status |

First step: open `/api/health`. If it is not JSON `{"status":"ok",…}`, the backend is not being served.

### Vercel shows `500 FUNCTION_INVOCATION_FAILED`

Open **Vercel → Project → Logs** and read the error:

| Log contains | Cause | Fix |
|---|---|---|
| `ERR_MODULE_NOT_FOUND` … `/var/task/backend/src/app` | Relative imports missing `.js` | Add `.js` extensions (see [ESM rules](#esm-rules-for-the-server-code-important)) |
| `ERR_IMPORT_ATTRIBUTE_MISSING` / "needs an import attribute of type json" | JSON import without `with {type:'json'}` | Add the attribute |
| `Cannot find package '…'` | Dependency missing | Check `package.json` and the install step |

### Data shows "Bundled snapshot" when it should be live

Jolpica could not be reached or validated within the request budget. Check:

- `DATA_OFFLINE` is `false` or unset
- `JOLPICA_BASE_URL` is `https://api.jolpi.ca/ergast/f1`
- Outbound HTTPS to `api.jolpi.ca` is allowed
- You are not being rate limited (429); raise `CACHE_TTL_SECONDS` and wait

Direct test:

```sh
curl "https://api.jolpi.ca/ergast/f1/2024/races.json?limit=1"
```

### AI page always shows the deterministic summary

Possible causes: no `GROQ_API_KEY`, invalid key, the configured model is retired or unsupported (the usual cause with the old default), rate limit, or a Groq timeout. Set a current `GROQ_MODEL`, confirm the key, and redeploy. `/api/health` shows `aiConfigured: true` once a key is present.

### Blank page or failed imports

Run `npm ci` then `npm run build`; use Node 22 or newer.

### Port 5173 already in use

Stop the other process, then rerun `start.bat` or `npm run dev`.

### Slow or broken 3D

Choose low quality, stop auto-spin, or enable reduced motion. WebGL needs a compatible browser and GPU. If scene creation fails, a text fallback is shown and all data pages remain usable. Screenshot and fullscreen depend on browser permissions.

### Playwright cannot launch

Run `npx playwright install chromium` on a network that can reach Playwright's browser CDN.

---

## Security and privacy

- **Secrets stay server-side.** `GROQ_API_KEY` is read only by the Express function. Never prefix it with `VITE_`; Vite bundles `VITE_` variables into public browser code.
- `.env`, `.vercel/`, `node_modules/` and test output are in `.gitignore`. If a key is ever exposed (screenshot, chat, public repo), revoke it in the Groq console and create a new one.
- Helmet sets security headers on `/api`; `vercel.json` sets additional global headers.
- All inputs are validated with Zod; request bodies are capped at 24 KiB.
- Rate limits protect the API per warm instance; use Vercel Firewall for stronger quotas.
- Errors return generic JSON messages without stack traces.
- No user accounts, cookies for tracking, analytics, ads or server-side storage of conversations. The browser stores only UI preferences (motion and 3D quality) and an intro-seen flag.

---

## Known limitations

- Cars and trophy are **conceptual procedural art**, not licensed manufacturer CAD or official trophy replicas. Airflow is illustrative, not CFD.
- Circuit outlines are **artistic**. Verified turn, DRS and elevation geometry and flyovers are not provided.
- Telemetry is limited to the **two bundled Bahrain 2023 qualifying laps**. No GPS racing lines, exact real-time playback or live feeds.
- The biography and records are scoped through 2024. Recent results and standings are provider-backed and timestamped.
- Career points in comparisons are race-only; use championship standings for totals including sprints and adjustments.
- Strategy Monte Carlo intervals are illustrative and uncalibrated.
- The AI uses server-selected structured context only. It has no web browsing, persistent history, streaming or multi-step tool use.
- Cache and rate-limit state are not shared across serverless instances.
- Seasons before 2021 lack bundled calendar, constructors, qualifying and sprint snapshots.
- The 2026 snapshot is a partial season.
- There is no live timing feed. The Race Center shows a "No live timing feed" badge.

---

## Attribution and licensing

- **Race data:** Jolpica (<https://github.com/jolpica/jolpica-f1>), an Ergast-compatible community API. Follow its terms and rate limits.
- **Telemetry:** prepared with FastF1 (<https://github.com/theOehrly/Fast-F1>) from F1 timing data. FastF1's software licence does not grant ownership of the underlying F1 data. **No broad data-redistribution licence is asserted.** Review the applicable source terms before public or commercial redistribution.
- **Fonts:** Barlow and Barlow Condensed via Fontsource, distributed under the SIL Open Font License.
- **3D art, trophy, circuit paths, favicon:** original programmatic interpretations; no third-party meshes, logos or video.
- **Photos:** only use images you have the right to publish and keep any required credits (see [Photos](#photos)).
- Add your own `LICENSE` file for the project code before open-sourcing. None is included by default.

Formula 1, F1, FIA, Red Bull and related marks belong to their respective owners.

---

## FAQ

**Do I need a Groq key?** No. Everything except Groq-written answers works without one. Without a key, the AI page returns a clearly labelled deterministic summary.

**Should the Groq model name be secret?** No. Only the API key is secret.

**Does the app work offline?** The frontend and bundled snapshots, telemetry and photos work without internet, but live Jolpica data and Groq do not. Set `DATA_OFFLINE=true` to force snapshots locally.

**Why does Race Center say "Bundled snapshot"?** The live provider call failed or timed out, so the app used saved data for that exact season. See [Troubleshooting](#data-shows-bundled-snapshot-when-it-should-be-live).

**How do I add new race results after a Grand Prix?** Run `npm run data:update -- <season>`, then `npm test`, `npm run build`, commit and redeploy. Live retrieval also picks up new results automatically while Jolpica is reachable.

**Why does the API return 503 for old seasons?** If Jolpica is down and there is no bundled snapshot for that season and table (for example the 2018 calendar), the app refuses to guess and returns an explicit 503.

**Can I host this somewhere other than Vercel?** The Express app can run on any Node 22 host (`npm run build`, then `npm run preview` serves `dist/` and the API on port 5173). A purely static host such as GitHub Pages or Netlify without functions will not work, because `/api/*` needs a server.

**Why is the data different from official F1 sources?** Jolpica is community-maintained and can lag or correct results. Treat this site as a fan archive, not an official source.
