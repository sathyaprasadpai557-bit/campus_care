# Campus Care AI Demo Guide

This guide is for a **local, disposable demo database**. Never run the reset command against a MySQL or production database. The sample records and their pictures are synthetic illustrations, not genuine maintenance evidence or model-validation examples.

## Start and prepare

1. The prepared presentation data lives separately in `backend/instance/campus_care_demo.db` and synthetic images in `backend/demo-uploads/`; it does not replace the existing local database. In a backend PowerShell terminal, set `$env:DATABASE_URL='sqlite:///campus_care_demo.db'` and `$env:UPLOAD_DIR='demo-uploads'` before running the API. Gallery selection is off by default. To enable it for this local demo only, set `$env:DEMO_MODE='true'` and `$env:ALLOW_GALLERY_UPLOAD='true'` in the backend terminal before starting Flask. The frontend then shows **Choose from Gallery (Demo Only)**. To return to camera-only behavior, set both flags to `false` (or remove them) and restart Flask. Configure an `OPENAI_API_KEY` if live photo screening should run. Set `PHOTO_RELEVANCE_CONFIDENCE_THRESHOLD=0.72` or another demo threshold. Without the key, photo relevance is marked unclear and the app offers retake or explicit admin review.
2. From `backend`, install requirements and start the API with `python app.py` after setting those environment variables. Keep both gallery flags false in production; gallery mode is intended only for a local demo.
3. In another terminal, from `frontend`, run `npm install` once and `npm run dev`.
4. Optional, only on the disposable presentation database: from `backend`, set `$env:DEMO_RESET_CONFIRM='YES'`; keep `DATABASE_URL=sqlite:///campus_care_demo.db` and `UPLOAD_DIR=demo-uploads`; run `python reset_demo_db.py`, then `python seed_demo_data.py`. The reset refuses every database except the exact isolated demo SQLite file and deletes all its records; it leaves uploaded files untouched.
5. Seeded local demo credentials on a fresh/reset database:
   - Admin: `admin@campus.edu` / `Admin123!`
   - Student: `student@campus.edu` / USN `STUDENT001` / `Student123!`
   - Worker accounts: `electrician@campus.edu`, `plumber@campus.edu`, `carpenter@campus.edu`, `cleaner@campus.edu`, `civil@campus.edu`, `it@campus.edu`, and `general@campus.edu`, initially `Worker123!`.
   - A fresh seeded student and workers are prompted to change temporary passwords. A reset restores these demo credentials. Do not use them outside local development.

## Three-to-five-minute live demo

### Student (about 1 minute)

1. Sign in as the demo student. The portal opens directly on **Register Complaint**.
2. Capture a real visible maintenance issue with the in-app camera. Enter a building, floor, room/area, a precise description, and the category you believe is correct.
3. Show the photo screening outcome and AI category/priority. The AI prediction is separate from the student-selected category; correct the selection if needed.
4. Submit and open **My Complaints**. Show complaint ID, date, category, description, location, status, and either the assigned worker and role or the pending-assignment message.
5. Open the complaint detail to show before/after evidence. The student confirms the repair with the QR verification; otherwise the student can reopen it.

6. To demonstrate cancellation, create another complaint and select **Delete Complaint** while it is still Submitted, AI Analyzed, Pending Assignment, or assigned to a worker who has not started the repair. Confirm the prompt. The complaint moves to **Cancelled history** for the student and remains visible to admins with its audit history. A complaint whose repair has started or is already submitted/resolved cannot be cancelled by the student; the UI directs them to contact the admin.

### Demo gallery selection (optional)

With both demo flags enabled on the backend, choose an image in the complaint form. The server still performs photo validation and rejects unsupported gallery uploads if either flag is off. Synthetic sample images are at `backend/demo-uploads/`; use them only to illustrate the flow. Cancellation is a soft status change: it preserves the complaint, uploaded evidence, repair record, and history rather than deleting database data.

### Invalid and unclear photo (about 30 seconds)

1. Capture an unrelated portrait, animal, food, scenery, meme, screenshot, or personal object. A person in frame is allowed when a real maintenance defect is also visible.
2. A confidently unrelated image shows: “Photo is not related to campus maintenance. Please capture a valid photo showing the issue.” Use **Retake Photo**; no complaint ID or worker assignment is created.
3. An unclear/obstructed image offers a clearer retake or explicit **Submit for admin review**. If submitted for review, it appears in **Admin → Flagged Photos** with no automatic worker assignment.

### Admin (about 1 minute)

1. Sign in as `admin@campus.edu`. Open **Pending Assignment**, **Flagged Photos**, and **All Complaints**.
2. Open a test complaint and show its real worker, role, AI recommendation, and audit history endpoint (`GET /api/admin/complaints/{id}/assignment-history`).
3. Choose **Remove Assignment**. Confirm the worker-specific dialog and optionally enter a reason. Verify the complaint remains, its status becomes **Pending Assignment**, and the assignment audit records the old worker, admin, timestamp, and reason.
4. Select another eligible worker or choose **Recommend worker**. Refresh and verify the updated worker name and role.

### Worker (about 1 minute)

1. Sign in as a worker assigned to the test complaint. The worker queue contains only complaints with that worker's ID.
2. Start the work, enter repair notes/materials, capture an after photo, and submit it for student verification.
3. Return to the student account and confirm the repair with the QR. The complaint becomes **Resolved**; admins can close it afterward.

## Troubleshooting checklist

- **Camera unavailable:** use `localhost` or HTTPS, grant browser camera permission, and refresh.
- **Every photo is unclear:** set a valid `OPENAI_API_KEY` in the ignored local `backend/.env`, restart Flask, and check API logs for a provider error. Never put the key in React/Vite variables.
- **Unrelated image reaches an admin queue:** inspect `GET /api/admin/photos/validation-audits`, then inspect the complaint under **Flagged Photos**. A confident unrelated result must produce HTTP 400 on submission; an unclear result is HTTP 409 until the student explicitly asks for review.
- **Worker isn't assigned:** check worker availability, job role/skills, `GET /api/admin/complaints`, and the complaint's `assigned_worker_id`. No eligible worker leaves the report pending.
- **Remove Assignment is denied:** the complaint must not be Closed; reopen it through the authorised workflow first.
- **No phone call/WhatsApp:** assignment still appears in the portal, but external delivery also needs Twilio credentials, configured sender IDs, worker phone numbers, and a public HTTPS backend callback.
- **Student cancellation is unavailable:** it is only offered for that student's own complaint before repair work starts. Once the worker starts, or repair is submitted/resolved, contact an administrator to request cancellation.
- **Demo accounts don't match:** reset only the disposable SQLite demo database; existing account passwords cannot be recovered from password hashes.
- **Check the APIs:** `/api/health`, `/api/complaints/student`, `/api/admin/complaints`, `/api/worker/assigned-complaints`, `/api/admin/photos/validation-audits`, and the remove/recommend/reassign endpoints.

## Photo test coverage and limits

The automated policy tests exercise valid examples (fan, tap, bench, switch, garbage, projector/computer), unrelated examples (selfie, portrait, person-only image, animal, food, landscape, meme, personal object), and low-confidence/obstructed outputs. These tests mock vision responses to verify allow/reject/review decisions; they do not measure a live model's recognition accuracy. Real image accuracy tests require a configured vision key and actual example photos. The model is a screening aid and cannot establish that a photo is authentic.

## Database and demo reset

On startup `db.create_all()` adds the `assignment_audits` table for existing installations; migrations also add cancellation metadata and normalize legacy `Assigned` rows plus inconsistent assignment IDs/statuses. Fresh MySQL installs can use the updated `backend/schema.sql`. The reset helper is locked to `campus_care_demo.db` and requires `DEMO_RESET_CONFIRM=YES`.
