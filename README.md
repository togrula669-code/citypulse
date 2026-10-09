# CITYPULSE AI v0.4.0 — Hackathon MVP

**English-language, full-stack JavaScript civic incident intelligence demo.** The browser and REST API are served by the same Node.js/Express process. No separate frontend dev server is needed.

Citizen text, photo, voice and short video are turned into **AI-assisted, evidence-aware incident reports**, checked for potential duplicates and reviewed in a prototype operator dashboard. This is **not a real municipal submission service**.

## Start on Windows (VS Code)

1. Extract the ZIP. Open the inner `citypulse-ai` folder in VS Code — the one containing `package.json`.
2. Install **Node.js LTS (20.19+ or 22+)** and open the VS Code terminal.
3. Run:

```powershell
npm install
Copy-Item .env.example .env
notepad .env
```

4. Put **your own** Groq key in `GROQ_API_KEY` in `.env`. Set `ADMIN_TOKEN` to a long secret value for operator access. Leave the AI key **out of frontend files**.
5. Validate account models, then start:

```powershell
npm run groq:models
npm run groq:test
npm run check
npm run dev
```

Open **http://localhost:3001**. If the model list reports that the configured vision model is unavailable, choose a vision-capable model that actually appears in your account and edit `GROQ_VISION_MODEL`. API limits, access and billing are governed by your provider account; no provider success is guaranteed without a live key test.

You can instead configure `AI_PROVIDER=gemini`, `GEMINI_API_KEY`, and a verified `GEMINI_MODEL`. Do **not** enable paid billing merely to run this demo; check model free-tier availability first.

### Video implementation — important truth about limitations

The browser decodes a short MP4/WebM video (up to **30 seconds** and **25 MB**) and captures **three actual image frames** locally. The backend sends them to the selected provider's **vision** endpoint (`POST /api/analyze/frames`). **The video's soundtrack is not analyzed.** To analyze speech, upload the audio independently. Video codec support depends on the user's browser. This avoids sending a potentially huge raw video to an unsupported Groq API.

This is *not* continuous video understanding, full motion tracking, or automatic audio-video synchronization. Gemini direct-video API remains available in the backend when a compatible account/model is configured, but the web UI intentionally uses bounded image-frame analysis for more predictable request sizes.

### Optional private local evidence archive

Default `EVIDENCE_STORAGE=none`: original evidence is **not saved** after analysis. Optional `EVIDENCE_STORAGE=local` enables a citizen-facing **explicit opt-in** to save the media (for video-frame analysis, only the *first representative frame*) under `data/evidence/`, protected by admin authentication. The operator can download archived files. No evidence upload is archived without opt-in. It has no automatic retention/deletion policy — delete private demo files manually. Do not enable this for real public deployments without privacy review.

The local file-backed database is under `data/db.json`. Data files are for **one-machine hackathon use**, not distributed cloud deployments. Supabase remains a proposed future adapter; it is **not implemented**. Never deploy on an ephemeral filesystem expecting durable persistence.

## Features

- Modern responsive citizen UI and prototype operator panel.
- Text → AI structured JSON; photo → vision analysis; audio → Groq Whisper and analysis when configured; short video → browser sampled vision frames.
- Separate citizen claims, AI inferences, observed evidence, missing information and qualitative confidence.
- Priority engine with distinct AI severity and operational priority.
- Duplicate candidates using location + Unicode token/synonym overlap + incident subtype. **Heuristic, not embeddings or automatic merging.**
- Baku-centered Leaflet/OpenStreetMap maps; coordinates require consent or manual selection. Public map positions are approximate.
- Single-use analysis tickets, public data redaction, private operator actions and optional protected evidence archive.
- File-backed incident/work-order persistence, synthetic demo seed, unit and HTTP tests.

## Development and tests

```powershell
npm run verify:frontend
npm test
npm run evaluate:duplicates
npm run diagnose
npm run diagnose:live
npm run http:check
npm run demo:seed
```

`npm test` contains automated unit tests plus a server HTTP integration test (requires `npm install`). `npm run evaluate:duplicates` uses **12 carefully constructed, synthetic pairs** only and is **not** a field accuracy benchmark. The live AI smoke test requires an actual provider key, internet and quota. `npm run demo:seed` inserts clearly labeled synthetic Baku records, not real citizen reports.

## Important API routes

| Method | Route | Purpose |
|---|---|---|
| GET | `/api/health`, `/api/health/dependencies` | Non-inference diagnostics |
| POST | `/api/analyze/text` | Text report |
| POST | `/api/analyze/image` | Image report |
| POST | `/api/analyze/audio` | Audio and transcription |
| POST | `/api/analyze/frames` | 1–3 actual JPEG video frames |
| POST | `/api/analyze/video` | Direct-video compatible provider only |
| POST | `/api/incidents` | New / duplicate / supporting report decision |
| GET | `/api/incidents` and `/api/incidents/:id` | Redacted public records or full admin records |
| GET | `/api/evidence/:id` | Admin-only private media retrieval |
| PATCH | `/api/incidents/:id/status` | Operator status workflow |
| GET/POST | `/api/work-orders` | Protected work orders |
| PATCH | `/api/work-orders/:id` | Work order status transition |

**Privileged requests:** supply `x-admin-token` from the private server `.env` value. The UI has an operator unlock form. Do not publish the token.

## Hackathon readiness / honesty

Review **`HACKATHON_AUDIT.md`** for testing coverage, limitations and critical pre-demo checklist. No measured classification accuracy is claimed unless actual labeled media was evaluated. Model access, media browser compatibility, and full live end-to-end requests must be tested on the competition laptop before presenting.


### v0.4.1 media upload fix

Media uploads now recognize valid MP3 frame variants and common browser MIME aliases. Real mismatch errors tell you the detected container and recommended conversion. See `TROUBLESHOOTING_TR.md` for steps. This fix is independent of your Groq/Gemini API key. Never store actual API credentials in this README; use `.env`.
