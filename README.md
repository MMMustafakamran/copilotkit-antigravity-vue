# Antigravity-vue: CopilotKit docs QA (Google Antigravity, Vue)

QA harness for the CopilotKit Vue docs at https://docs.copilotkit.ai/vue/google-antigravity. Every doc page gets a route in a real Vite + Vue app with the doc's snippets pasted **verbatim**, and a screen-recorded clip shows what happens when a user follows the page. Where the docs break, the clip shows the break and the defect is listed in [FINDINGS.md](FINDINGS.md). Nothing here fixes a doc defect; the only allowed edits are marked `HARNESS` (rules: `.qa/instructions/QA-RULES.md`).

Status: recorded 2026-11-04, findings logged. **The clips need re-recording** (the old recorder drew red outlines on the page). 3 pages tracked in [assignment.json](assignment.json), 2 reachable (❌ Quickstart, ✅ Frontend Tools; Chat Components is marked `reachable: none` and kept out of the report), 2 clips, both on Loom. Current report: [report/status.html](report/status.html).

## Layout

```
backend/                 own copy of the doc's Antigravity quickstart agent (Python, uv), 8141
my-copilot-app/          Vite + Vue app, one route per doc page; runtime server.antigravity.ts (8140)
doc-snapshot/            rendered / .md snapshot of the docs (2026-11-04)
autorecorder/            clip recorder: cli.ts → core/engine.ts → actions/*.ts; config/*.config.ts; output videos/ (not committed)
assignment.json          per-page status, notes, findings, clip ids, Loom links
report/status.html       generated from assignment.json (paste-friendly, team Google Doc format)
FINDINGS.md              open doc defects (stable #N ids)
.qa/                     vendored shared tooling + rules (synced; don't edit here)
start-all.sh             starts the backend, runtime and Vite on the harness ports (see Run)
```

## Run

This repo runs on its own: it has its own Antigravity quickstart agent in `backend/` and does not need Antigravity-react.

Prerequisites: Node.js 20+, Python 3.13 with [uv](https://docs.astral.sh/uv/), Git Bash (or any bash), `curl`.

1. Key: copy `.env.example` to `backend/.env` and fill in `GEMINI_API_KEY` (gitignored).
2. Install: `cd backend && uv sync`, then `cd ../my-copilot-app && npm ci`.
3. Start everything: `./start-all.sh` (logs in `logs/`). It starts, in order:

| Port | Server |
|---|---|
| 8141 | `backend/main.py`: the doc's Antigravity quickstart agent (`uvicorn main:app`, health at `/health`) |
| 8140 | `my-copilot-app/server.antigravity.ts`: runtime, HttpAgent -> :8141 |
| 5140 | Vite app: http://localhost:5140 |

Notes:
- Point the runtime at another agent with `AGENT_URL=... ./start-all.sh`.
- Never run on 3000 / 8000.
- `backend/` is copied verbatim from the docs' quickstart (via Antigravity-react/backend); only the port is a harness change, set in `start-all.sh`.
- Record clips: `cd autorecorder && npm run record` (see `autorecorder/`).
## How to review and report

- **A clip**: it should go doc → IDE (verbatim code) → demo, and for ❌/⚠️ also evidence (the faulty doc text drag-selected) → Notepad note. Nothing may be injected on the page.
- **A status**: ✅ works as documented, ⚠️ main feature works with a minor defect, ❌ main feature doesn't work as documented. If in doubt, ❌.
- **A finding**: add it to `FINDINGS.md` with its evidence (probe result, error text, frame). Format: QA-RULES section 5.
- **The report**: edit `assignment.json` (schema: `.qa/report/SCHEMA.md`), then rebuild. Don't hand-edit the HTML. Rows use the team doc's older Antigravity `status` style (`report.style`).

  ```sh
  python .qa/report/build.py assignment.json --out report/status.html
  # framework report for the team doc (React file first; needs a checkout of Antigravity-react beside this repo)
  python .qa/report/build.py ../Antigravity-react/assignment.json assignment.json --out Antigravity-status.html
  ```
