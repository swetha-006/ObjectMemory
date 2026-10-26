# Deploy ObjectMemory on Render Free (SQLite demo)

This configuration deploys the Vite frontend and Express API together as a single **free Render Web Service**. It keeps the existing SQLite database and local uploads implementation, but **Render Free has an ephemeral filesystem**. This is suitable only for a portfolio/demo environment using disposable test data.

## Critical data-loss warning

On Render Free, local files are lost whenever the service redeploys, restarts, or spins down after 15 minutes without inbound traffic. This includes `data/objectmemory.sqlite`, SQLite WAL files, and files under `uploads/`. User accounts, objects, timeline entries, uploaded photos/documents, and other locally stored records can disappear. A sleeping service may take about a minute to wake up. Do not store real personal documents or rely on this deployment for durable storage.

The app still uses SQLite; no database migration is performed. All existing features remain in the code, but anything that depends on saved database rows or uploaded files is only as durable as the current free-service filesystem. Optional API integrations continue to work when their keys are configured.

## Before deploying

- Do not commit `.env`, `data/`, `uploads/`, `node_modules/`, or real user data.
- If a real SerpApi key was ever committed or shared, revoke/rotate it. `.env.example` contains placeholders only.
- Do not use real receipts, identity documents, private photos, or confidential records on this demo deployment.
- `ENABLE_DEMO_ACCOUNT=false` disables automatic public demo-account creation in production.

## Deploy with the Blueprint

1. Commit and push this project to GitHub.
2. In Render, choose **New → Blueprint** and select the repository containing `render.yaml`.
3. Confirm the service plan is **Free**. The Blueprint intentionally defines no persistent disk and does not set `/var/data` paths.
4. Apply the Blueprint. It creates the `objectmemory` web service and generates `OBJECTMEMORY_JWT_SECRET`.
5. Optional: add `GROQ_API_KEY` and `SERPAPI_KEY` under the Render service's **Environment** page. Never commit these values. Without Groq, the built-in assistant fallback remains available. Without SerpApi, market refreshes cannot retrieve new Google Shopping results.
6. Wait for deployment to finish. Open `https://YOUR-SERVICE.onrender.com/api/health`; expect JSON with `status: "healthy"`.
7. Open the base service URL and test with a throwaway account and sample data only.

## Manual Render settings (if not using the Blueprint)

- Create a **Web Service** and select the **Free** instance type.
- Runtime: Node
- Branch: `main` (or your chosen branch)
- Build command: `npm ci && npm run build`
- Start command: `npm start`
- Health check path: `/api/health`
- **Do not add a disk** for this free deployment.
- Environment variables:
  - `NODE_ENV=production`
  - `OBJECTMEMORY_JWT_SECRET=<random secret, at least 32 characters>` (or use the Blueprint-generated value)
  - `ENABLE_DEMO_ACCOUNT=false`
  - `GROQ_MODEL=qwen/qwen3.8-27b`
  - `MARKET_GL=in`, `MARKET_HL=en`, `MARKET_GOOGLE_DOMAIN=google.co.in`
  - Optional: `GROQ_API_KEY`, `SERPAPI_KEY`

Do **not** set `OBJECTMEMORY_DATA_DIR=/var/data` or `OBJECTMEMORY_UPLOAD_DIR=/var/data/uploads` on this free configuration. With those variables omitted, the app uses its default `data/` and `uploads/` folders on the ephemeral filesystem.

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
- [ ] Register a throwaway user and log in.
- [ ] Create, edit, search, and delete a test object.
- [ ] Upload a sample object photo and a sample document.
- [ ] Add an incident/timeline entry.
- [ ] Test per-object chat and archive copilot (fallback works without Groq).
- [ ] Test dossier generation and JSON/CSV export/import using non-sensitive test data.
- [ ] If SerpApi is configured, test market refresh.
- [ ] Understand that the data may be lost after idle spin-down, restart, or redeployment.

## Upgrade path

To retain SQLite and uploads across restarts, use a paid Render web service with a persistent disk and set `OBJECTMEMORY_DATA_DIR=/var/data` and `OBJECTMEMORY_UPLOAD_DIR=/var/data/uploads`. A more scalable long-term design would use a managed database plus persistent/object storage.
