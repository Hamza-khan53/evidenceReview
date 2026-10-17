Evidence Review: Case Desk (extended fork)

A full-stack claim-review app. Pick an object type, describe the claim, upload photos, and get a structured verdict (supported, contradicted or not enough information) with a confidence score. Uncertain cases go to a human review queue, and every case is saved to a searchable history.

Attribution: this is a fork of Evidence Review by UPSTREAM_AUTHOR. The original provides the claim-adjudication pipeline, the FastAPI backend and the case-file UI. Everything listed under What this fork adds was built on top of it. See the upstream repository for its license terms.

What this fork adds
Feature	Details
Confidence scoring	The model returns a 0–1 confidence, clamped and validated server-side, shown as a colour-coded meter
Human review queue	Verdicts below MANUAL_REVIEW_THRESHOLD (default 0.6) or flagged manual_review_required are marked pending. A reviewer can approve, reject or request more info, with a note
Persistent case history	Every submission is stored in SQLite. A Case History tab shows stats, filters by verdict and review state, and reopens any past case
Input validation	Image type, size (8 MB) and count (8) limits, claim length limit, object type check
Security fixes	User text is escaped in the UI, and the /api/debug endpoint that exposed server paths was removed
Project structure
pipeline/            # shared adjudication logic
  schema.py          # enums, validation, confidence helpers
  prompts.py         # system prompt and user prompt builder
  io_utils.py
backend/
  app.py             # FastAPI app: claims, history, review, stats
  store.py           # SQLite persistence
  requirements.txt
frontend/
  index.html
  styles.css
  app.js
Run locally
bash
cd backend
pip install -r requirements.txt
export ANTHROPIC_API_KEY=sk-...
uvicorn app:app --reload --port 8000
# open http://localhost:8000 (one process serves the UI and the API)

On Windows PowerShell, use $env:ANTHROPIC_API_KEY="sk-..." instead of export.

Configuration
Variable	Default	Purpose
ANTHROPIC_API_KEY	none (required)	Enables vision adjudication
CLAIM_REVIEW_MODEL	claude-sonnet-4-6	Model used for verdicts
EVIDENCE_DB	backend/claims.db	Path of the SQLite database
FRONTEND_DIR	../frontend	Where static files are served from
API
Method	Endpoint	Description
POST	/api/claims	Submit a claim (multipart: claim_object, user_claim, files) and get a verdict
GET	/api/claims	List cases. Optional claim_status, review_status, limit
GET	/api/claims/{id}	One case with its full result
POST	/api/claims/{id}/review	Record a reviewer decision: approved, rejected or needs_more_info, plus an optional note
GET	/api/stats	Totals, counts by verdict, pending reviews, average confidence
GET	/api/health	Server status and whether the API key is configured
Deploy (Render, free tier)
Push the project to a GitHub repo.
On render.com choose New → Blueprint and connect the repo. Render reads render.yaml.
Set ANTHROPIC_API_KEY in the Render dashboard. It is never committed.
Deploy. One URL serves both the UI and the API.

A Procfile is included for Railway or Heroku-style platforms.

Free-tier caveats: services sleep after about 15 minutes idle and take 30–60 s to wake. The disk is ephemeral, so case history resets on each redeploy. Point EVIDENCE_DB at a persistent disk, or swap store.py for Postgres, to keep it.

Known limitations
Uploaded images are not stored, only the verdict and metadata.
evidence_requirements.csv and user_history.csv lookups are not wired in, so no per-user risk context is passed to the model.
No authentication or rate limiting, and CORS is open (*). Fine for a demo, not for public use.
Confidence is the model's self-reported estimate and has not been calibrated against labelled data.
Possible next steps
Store uploaded images in object storage (S3/GCS) and show them in case history.
Move persistence to Postgres.
Add authentication and tie claims to users, then wire in user-history risk context.
Evaluate confidence calibration on a labelled set of claims.
Restrict CORS to the real frontend origin and add rate limiting.
