# IEEE Impact++ — Growth & Evidence MVP

A full-stack starter for Christina's Growth & Evidence module.

- **Frontend:** static HTML/CSS/JavaScript, deployable to GitHub Pages.
- **Backend:** FastAPI + SQLAlchemy, deployable separately (Render, Railway, Fly.io, etc.).
- **Database:** SQLite locally; PostgreSQL in production through `DATABASE_URL`.
- **AI:** OpenAI API called only by the backend. Never put the API key in frontend code.
- **Student experience:** one student-facing experience; peer reviewers are simulated by the backend.

## What's implemented

- Student registration/login with JWT authentication.
- Career selection and seeded role/skill catalog.
- Curated Moves across Explore, Create, Test, Share, Connect, Enable.
- Evidence submission as a written summary, URL, or uploaded file.
- AI evidence rating against four equally weighted criteria.
- Automated simulated peer review, reviewer eligibility, no self-review, reciprocal-review guard, daily review cap.
- Impact Score, validated/effort split, weekly effort cap, banked effort, diminishing returns, score ledger.
- Skill tiers, role tier, opportunity unlocks, badges, growth state.
- Dashboard, career roadmap, Moves, evidence, score ledger, badges, opportunities, profile pages.

## Important scope notes

1. Uploaded files are stored on the backend filesystem in this starter. For production, use object storage (S3/R2) and malware scanning.
2. URL evaluation does **not** crawl arbitrary websites. The AI receives the submitted URL and student summary; it must not claim to have inspected inaccessible link contents. Add a trusted fetcher/connector only if you want actual repository or webpage ingestion.
3. Simulated reviewers are clearly labelled as simulated. This is a capstone demonstration workflow, not real human peer validation.
4. The supplied specification says a second review is required when `abs(AI score - reviewer score) > 0.35`. The starter implements this by assigning a second simulated reviewer.
5. The spec's scoring formula and rules are implemented in `backend/app/scoring.py`. Review the open items with teammates before integrating external modules.
6. This is a development-ready MVP scaffold, not a security-audited production service. Before real public launch, add email verification, password reset, rate limiting, stronger file controls, privacy policy, monitoring, and backups.

## Run locally

### 1. Backend

Python 3.11+ recommended.

```bash
cd backend
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate
pip install -r requirements.txt
copy .env.example .env   # Windows
# cp .env.example .env   # macOS/Linux
```

Edit `.env` and set `OPENAI_API_KEY`. `OPENAI_MODEL` defaults to `gpt-4.1-mini`.

```bash
uvicorn app.main:app --reload
```

API docs: http://127.0.0.1:8000/docs

### 2. Frontend

Serve the frontend folder with any static server. For example:

```bash
cd frontend
python -m http.server 5500
```

Open http://127.0.0.1:5500

Set the API base URL in `frontend/config.js`. For local use:

```js
window.IMPACT_API_BASE = "http://127.0.0.1:8000";
```

### 3. Create a student account

Use the Register screen in the app. The first registered user is automatically the demo student. The app does not seed a default password.

## Deploy backend

Example: Render.

1. Push this repository to GitHub.
2. Create a new **Web Service** from the repository.
3. Set the root directory to `backend`.
4. Build command: `pip install -r requirements.txt`
5. Start command: `uvicorn app.main:app --host 0.0.0.0 --port $PORT`
6. Add environment variables:
   - `OPENAI_API_KEY` = your secret OpenAI API key
   - `OPENAI_MODEL` = `gpt-4.1-mini` (or a model available to your account)
   - `JWT_SECRET` = a long random secret
   - `DATABASE_URL` = PostgreSQL connection string from your managed database
   - `FRONTEND_ORIGINS` = your GitHub Pages origin, e.g. `https://YOUR-USERNAME.github.io`
   - `UPLOAD_DIR` = `/opt/render/project/src/uploads` (or a persistent disk path)
7. Attach a persistent disk if you keep uploads on local disk. Prefer object storage for production.

The backend creates tables on startup for this MVP. Use Alembic migrations before production.

## Deploy frontend to GitHub Pages

1. Set `window.IMPACT_API_BASE` in `frontend/config.js` to your deployed backend URL, e.g. `https://your-service.onrender.com`.
2. Push the repository to GitHub.
3. In GitHub: **Settings → Pages → Build and deployment → Deploy from a branch**.
4. Choose `main` and `/frontend` if your Pages setup supports a folder source. Otherwise use GitHub Actions or publish the contents of `frontend/` from a `gh-pages` branch.
5. If the app is hosted at `https://YOUR-USERNAME.github.io/REPO/`, set `window.IMPACT_BASE_PATH = "/REPO/"` in `frontend/config.js` if you later add client-side deep routes. Current navigation uses hashes and does not require rewrite rules.

## Environment variables

See `backend/.env.example`.

## Tests

```bash
cd backend
pip install -r requirements.txt
pytest
```

## Scoring assumptions to verify with the capstone team

- `a` and reviewer score `r` are represented on a 0–1 scale.
- The four AI criteria are equally weighted.
- A review rating from 1–5 is normalized as `(rating - 1) / 4`.
- For AI confidence below 0.5, the formula uses `q = 0.2*a + 0.8*r`; otherwise `q = 0.4*a + 0.6*r`.
- Reviewers with Proficient tier weight 0.8, Demonstrated weight 1.0, and Verified/mentor weight 1.2.
- Effort points: evidence 3, learning module 4, review 2, active day 1, self-reported progress 1; raw weekly cap 20.
- The initial allowance is `max(30, 3/7 * V_total)` for counted effort.
- The role tier is based on at least 60% (rounded up) of core skills reaching a tier.
