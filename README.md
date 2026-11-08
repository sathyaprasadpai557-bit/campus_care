# Campus Care AI

A maintenance-only campus complaint prototype with role-based Student, Admin, and Worker portals. The app uses React/Vite, Flask, JWT, and MySQL (with SQLite as a local development fallback). AI features are transparent heuristics and explainable scoring, not a trained model.

## Run locally

1. Backend: `cd backend`, create/activate a virtual environment, then `pip install -r requirements.txt`.
2. For MySQL, run `backend/schema.sql`, then set `DATABASE_URL=mysql+pymysql://USER:PASSWORD@localhost/campus_care` in `backend/.env`. The default is SQLite for local prototyping.
3. Run `python app.py`. The API seeds a demo admin, student, and workers on first launch.
4. Frontend: `cd frontend`, run `npm install`, then `npm run dev`.

Demo credentials: `admin@campus.edu / Admin123!`, `student@campus.edu / STUDENT001 / Student123!`, `electrician@campus.edu / Worker123!`. The student account requires a password change on first login. Change all demo credentials before deployment.

Camera capture requires localhost or HTTPS and browser permission. Gallery selection is hidden by default and can be enabled only for a local demo by setting both `DEMO_MODE=true` and `ALLOW_GALLERY_UPLOAD=true` in the backend environment; keep both false in production. Student cancellation is available only before repair work starts and preserves a cancelled record and its evidence for admin history. The API stores image data locally under `backend/uploads`; configure production object storage and HTTPS before deployment.

## Optional integrations

Set `OPENAI_API_KEY` in `backend/.env` for separate maintenance-photo relevance screening, image categorization and repair-photo comparison. Configure `PHOTO_RELEVANCE_CONFIDENCE_THRESHOLD` (default `0.72`). Confident unrelated photos are rejected before a complaint row is created; unclear photos require a retake or explicit admin review. Without the API key, the outcome is unclear and is never silently accepted. Set Twilio credentials, sender IDs, registered worker phones, and publicly reachable `APP_PUBLIC_URL` and `BACKEND_PUBLIC_URL` for WhatsApp and voice calls. Without those settings, assignment still appears in the Worker Portal while delivery is recorded as failed. Keep secrets in the ignored local `backend/.env`; never put them in frontend variables.

See [DEMO_GUIDE.md](DEMO_GUIDE.md) for role-based demo steps, local demo credentials, troubleshooting and the SQLite-only reset instructions.

## Deploy a public demo and keep it updated

The root `Dockerfile` and `render.yaml` define a single-service Render deployment: Flask serves the built React app and API from one HTTPS origin, with a separate PostgreSQL database. To deploy, push this project to a private GitHub repository, sign in to Render, create a **Blueprint** from that repository, and enter a bootstrap admin email and a unique password of at least 12 characters when prompted. The deployment deliberately starts without demo student/worker accounts or demo credentials. Register accounts from the admin portal. The generated public URL can then be opened on a phone from any network. Render can automatically redeploy the linked service after each push to its connected branch.

The included Blueprint uses free preview resources: the web service may sleep when idle, the free PostgreSQL database expires after 30 days, and uploaded photos are stored on temporary service storage. Use this only for a short-lived demo with synthetic information. Do not enter real student records, phone numbers, or maintenance photos. A long-running campus deployment needs paid persistent database storage and durable private object storage for images, plus a review of authentication, access control, backups, and notification-provider secrets. Gallery uploads remain disabled in the deployed configuration.

## Prototype scope

Includes student complaint intake, duplicate linking/override, explainable worker matching, audited admin assignment removal/recommendation/reassignment, admin account and complaint management, worker updates and after-photo completion, student repair confirmation/reopen, ratings, basic progression recommendations, and SQL schema. Visual screening can flag low-quality, reused, near-duplicate, metadata, or model-indicated suspicious images for review; it cannot prove that an image is authentic or edited. GPS is optional and never used as a fraud signal.
