# NovaWorks AI Project Manager — The Infinity Hack '26

**Team:** _<your team name>_  ·  **Challenge:** AI Project Manager – Meeting to Execution

A desktop CRM (Windows `.exe` or any browser) for NovaWorks Technologies. The admin pastes a meeting
transcript, an LLM turns it into projects and tasks (managers, developers, deadlines, estimated hours),
and role-based screens show each person only what they are allowed to see.

## Stack
Electron 33 · Node/Express · SQLite (sql.js, persisted to a file) · bcryptjs · Claude API (Anthropic) or Google Gemini · vanilla JS frontend

## Working features
- Login/logout with the 10 supplied demo accounts (session cookie; role always taken from the server session)
- Admin: all projects, team directory, **Create from Transcript**, settings
- Manager: only their projects. Agent: only their own tasks + the related project (never other agents' tasks, counts or hours)
- Projects list/detail, task rows (title, description, agent, deadline, hours), dashboard + workload chart
- AI conversion: loading state, success summary, understandable errors; invalid/unresolved output saves **nothing** and shows an editable draft to correct and revalidate
- All-or-nothing database transaction; double-click / concurrent submits blocked (UI + server)
- Data persists across restarts; seeder is idempotent (no duplicate users)
- Access rules are enforced in the API, not just hidden in the UI (direct requests to other people's projects return 404)

## Run the .exe (easiest)
1. Double-click `NovaWorks-AI-PM.exe` (portable, no install). Windows SmartScreen may warn because the file is unsigned: **More info → Run anyway**.
2. Sign in as `admin@novaworks.example` / `Demo123!`
3. **Settings** → paste an API key → Save (stored locally, never sent to the UI). Use either a **free Google Gemini key** (`AIza…`, from https://aistudio.google.com/apikey) or an Anthropic key (`sk-ant-…`); the provider is detected automatically.
   Alternative: put a `.env` file (see `.env.example`) in the same folder as the .exe.
4. **Create from Transcript** → *Insert supplied sample* (or paste your own) → **Create from Transcript**.

Data lives in `%APPDATA%\NovaWorks AI Project Manager\` (`novaworks.sqlite`, `config.json`).
Settings → *Clear all projects & tasks* resets the demo (users are kept).

## Run from source
```bash
npm install
cp .env.example .env        # add ANTHROPIC_API_KEY
npm run seed                # creates data/novaworks.sqlite with the 10 demo users (safe to re-run)
npm run server              # browser: http://localhost:3000
npm start                   # or: desktop window via Electron
npm test                    # end-to-end checks (uses a fake AI endpoint, no key needed)
npm run dist                # builds dist/NovaWorks-AI-PM.exe (portable, Windows x64)
```
The desktop app seeds the demo users automatically on every start.

## Environment variables
`ANTHROPIC_API_KEY` (holds either key type; a key saved in Settings overrides it) · `AI_MODEL` (defaults: `claude-sonnet-5-5` / `gemini-2.5-flash`) · `PORT` (browser mode) · `DATA_DIR` (browser mode DB folder)

## Demo accounts (fictional) — password for all: `Demo123!`
| Role | Email |
|---|---|
| Admin | admin@novaworks.example |
| Manager | ayesha@ / bilal@ / hina@novaworks.example |
| Agent | ali@ / hamza@ / sara@ / usman@ / zain@ / maryam@novaworks.example |

## Testing the transcript flow
1. Admin → Create from Transcript → *Insert supplied sample* → Create. Expect **3 projects, 12 tasks** (UrbanCart 40 h, QuickServe 46 h, HelpDeskPro 38 h).
2. Ayesha → only UrbanCart. Ali → only his 3 tasks. Hamza → 2 tasks across UrbanCart and QuickServe.
3. Refresh/restart → data is still there.
4. Changed-input test: Settings → clear data, edit the transcript (e.g. QuickServe integration → 12 hours, 23 October), convert again → only that task changes.
5. Unresolved case: remove the final-recap lines naming a manager/assignee → AI/validation reports the problem and saves nothing; fix it in the draft editor and revalidate.

## How the AI step works
`server/ai.js` sends the transcript plus a directory of existing users (id, name, role, skills — **no emails or passwords**) to Claude with rules to follow final decisions, ignore rejected features, and never invent people. `validateDraft` then checks every project/task (existing MANAGER/AGENT, valid dates, task date ≤ project date, positive hours) before one transaction saves everything. App code generates all IDs.

## Deployment
Local desktop build (no hosted DB). For the optional hosted bonus, the Express app in `server/` can be deployed to any Node host; swap `server/db.js` for PostgreSQL (e.g. Aiven free tier).

## Known limitations
- Requires internet + an API key (free Gemini key works) for transcript conversion (no offline fallback by design: output must come from the AI).
- The .exe is unsigned and Windows-only; it was cross-built on Linux, so run it once on the demo machine beforehand.
- Editing created projects/tasks is not implemented (optional in the brief); unresolved drafts are corrected as JSON.
