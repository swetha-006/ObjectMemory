# Deploy ObjectMemory on Render (SQLite)

This deployment serves the Vite frontend and Express API from one Render Web Service. SQLite and uploaded files are stored on the persistent disk mounted at `/var/data`.

## Before deploying

- Review `.gitignore`: local `data/`, `uploads/`, and `.env` files must not be committed.
- If a real SerpApi key was ever committed or shared, revoke/rotate it. `.env.example` contains placeholders only.
- Do not put real users' receipts, photos, database files, or documents in the public Git repository.
- Do not enable `ENABLE_DEMO_ACCOUNT` in production.

## Deploy with the Blueprint

1. Push this project folder to a GitHub repository. Do not push `data/`, `uploads/`, or `.env`.
2. In Render, choose **New → Blueprint** and select the repository containing `render.yaml`.
3. Review the service plan. The `starter` web service plan is intentional because Render persistent disks require a paid service plan; a free web service does not provide persistent disk storage.
4. Apply the Blueprint. It creates the `objectmemory` web service, a 1 GB persistent disk at `/var/data`, and a generated `OBJECTMEMORY_JWT_SECRET`.
5. Optional: add `GROQ_API_KEY` and `SERPAPI_KEY` under the Render service's **Environment** page. Never commit these values. Without Groq, the built-in assistant fallback remains available. Without SerpApi, market refreshes cannot retrieve new Google Shopping results.
6. Wait for the deploy to finish. Open `https://YOUR-SERVICE.onrender.com/api/health`; expect JSON with `status: "healthy"`.
7. Open the base service URL. Register a new account and test the workflows in the checklist below.

## Manual Render settings (if not using the Blueprint)

- Runtime: Node
- Branch: `main` (or your chosen branch)
- Build command: `npm ci && npm run build`
- Start command: `npm start`
- Health check path: `/api/health`
- Add a persistent disk mounted at `/var/data` (1 GB to start; disk-backed service is paid).
- Environment variables:
  - `NODE_ENV=production`
  - `OBJECTMEMORY_DATA_DIR=/var/data`
  - `OBJECTMEMORY_UPLOAD_DIR=/var/data/uploads`
  - `OBJECTMEMORY_JWT_SECRET=<random secret, at least 32 characters>`
  - `ENABLE_DEMO_ACCOUNT=false`
  - `GROQ_MODEL=qwen/qwen3.8-27b`
  - `MARKET_GL=in`, `MARKET_HL=en`, `MARKET_GOOGLE_DOMAIN=google.co.in`
  - Optional: `GROQ_API_KEY`, `SERPAPI_KEY`

## Local verification before pushing

```powershell
npm ci
npm run build
$env:NODE_ENV = "production"
$env:OBJECTMEMORY_JWT_SECRET = "replace-with-a-random-secret-longer-than-32-characters"
$env:OBJECTMEMORY_DATA_DIR = "$PWD\data"
$env:OBJECTMEMORY_UPLOAD_DIR = "$PWD\uploads"
npm start
```

Open `http://localhost:5000` and `http://localhost:5000/api/health`. For a clean production-mode test, use a disposable local data directory, not your real database.

## Post-deploy smoke test

- [ ] Homepage loads and browser console has no fatal errors.
- [ ] `/api/health` returns healthy JSON.
- [ ] Register a new user and log in.
- [ ] Create, edit, search, and delete a test object.
- [ ] Upload an object photo and a document; reload and confirm they still appear.
- [ ] Add an incident/timeline entry and verify it persists.
- [ ] Test per-object chat and archive copilot (fallback works without Groq).
- [ ] Test dossier generation and JSON/CSV export/import using non-sensitive test data.
- [ ] If SerpApi is configured, test market refresh.
- [ ] Restart/redeploy and confirm test records and uploads persist.

## SQLite operational notes

- Keep one running service instance. Render disks are attached to one instance and cannot be shared by multiple instances; SQLite is not suitable for horizontal multi-instance scaling on this setup.
- Back up the SQLite database and uploads before major changes. Do not copy only the main `.sqlite` file while the database is live in WAL mode; use SQLite's backup API or a consistent stopped-service backup.
- A persistent disk protects against routine redeploy/restart loss, but it is not a substitute for off-platform backups.
