# DATANIST Deployment Guide (Vercel)

This project is a Flask app deployed as a Vercel Python Serverless Function (`index.py`).

## Why your current deploy crashes

If Vercel shows **`FUNCTION_INVOCATION_FAILED`**, the most likely root cause was write attempts in a read-only project directory during cold start. The app now uses a writable runtime directory (`/tmp/datanist` on Vercel) for:

- uploads (`/tmp/datanist/uploads/cv`)
- mutable JSON runtime copy (`/tmp/datanist/seed_data.json`)

> Note: `/tmp` is ephemeral. Data survives only for the life of an instance, not permanently.

---

## 1) Required project structure

Ensure these files exist at repo root:

- `index.py` (Vercel entrypoint)
- `vercel.json`
- `requirements.txt`
- `app/app.py`
- `app/__init__.py`

---

## 2) Environment variables in Vercel

In **Project → Settings → Environment Variables**, set:

- `SECRET_KEY` (required)
- `GEMINI_API_KEY` (optional)
- `RAPIDAPI_KEY` (optional)
- `FLASK_ENV=production` (optional)

Optional runtime overrides:

- `DATANIST_RUNTIME_DIR=/tmp/datanist`
- `DATANIST_DATA_FILE=/tmp/datanist/seed_data.json`
- `DATANIST_UPLOAD_DIR=/tmp/datanist/uploads/cv`

---

## 3) Deploy steps

### Option A: Git integration (recommended)
1. Push this repo to GitHub/GitLab/Bitbucket.
2. In Vercel, import the repo.
3. Framework preset: **Other**.
4. Root directory: repository root.
5. Build command/output directory: leave empty.
6. Add env vars and deploy.

### Option B: Vercel CLI
```bash
npm i -g vercel
vercel login
vercel
```

```
---

## 4) Validate deployment

1. Open production URL.
2. Check homepage loads (no function crash page).
3. In Vercel Deployment logs, verify no startup `PermissionError` / read-only filesystem errors.
4. Test key routes:
   - login
   - student dashboard
   - mentor dashboard

---

## 5) Important limitation (must fix for real production)

Current app state is JSON-file based. On Vercel serverless this is not durable storage.

- Writes to JSON/upload files are ephemeral.
- Data can disappear after cold starts/redeploys/instance changes.

For production correctness, migrate persistence to a database/storage service:

- Postgres (Neon/Supabase/RDS)
- Vercel KV / Redis
- MongoDB Atlas

---

## 6) Quick troubleshooting

- **500 `FUNCTION_INVOCATION_FAILED` immediately**
  - Check logs for import/runtime filesystem errors.
  - Confirm requirements installed and `index.py` import path valid.

- **Module import errors**
  - Ensure every dependency is pinned in `requirements.txt`.

- **Static files not loading**
  - Verify `/static` route in `vercel.json` maps to `app/static`.

- **Login works but updates are lost**
  - Expected with ephemeral runtime file storage. Use external DB.