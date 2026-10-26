# ReelAnalyzer

A full-stack app that analyzes short videos (reels). Give it a video URL or upload a video file, and the backend downloads or saves the video, extracts its audio, transcribes it with a local Whisper model, and reports transcript and signal metrics. You can then chat with the transcript through a Gemini-powered chat endpoint. Progress is streamed back to the frontend as server-sent events.

## How it works

1. The frontend sends a video URL (`POST /api/analyze/url`) or an uploaded file (`POST /api/analyze/upload`) to the FastAPI backend.
2. The backend downloads the video with yt-dlp (for a URL) or saves the uploaded file, and copies it into `backend/media/` so the frontend can play it back.
3. Audio is extracted with MoviePy, then `backend/ai_service.py` transcribes it with local Whisper and computes metrics.
4. Progress updates stream back as server-sent events (`progress` events from 10% to 100%, then a `complete` event with the result, or an `error` event).
5. The chat endpoint (`POST /api/chat`) answers questions about a transcript using Gemini.

## Tech stack

- Frontend: React 19, Vite 5, Tailwind CSS, Framer Motion (`frontend/package.json`), with a `netlify.toml` for Netlify deployment.
- Backend: Python FastAPI (`backend/main.py`, titled "ReelAnalyzer SSE API"), served with Uvicorn.
- Video/audio: yt-dlp for downloads, MoviePy for audio extraction.
- AI: local Whisper for transcription (`backend/ai_service.py`) and Gemini for transcript chat.
- Backend files: `backend/main.py`, `backend/ai_service.py`, `backend/requirements.txt`, `backend/Dockerfile`, `backend/COOKIES_GUIDE.md`.

## API endpoints

- `POST /api/analyze/url` — analyze a video from a URL (SSE stream).
- `POST /api/analyze/upload` — analyze an uploaded video file (SSE stream).
- `POST /api/chat` — chat about a transcript (`transcript_text` and `question` in the request body).
- `GET /api/health` — returns `{"status": "ok"}`.
- `GET /media/...` — serves processed videos for frontend playback.
- When the frontend is built, the backend also serves it (including SPA routing); without a build, `/` reports that the frontend is not found and to build it first.

## How to run

The `start.sh` script builds the frontend and starts the backend on port 7860:

```bash
bash start.sh
```

That script runs `npm install` and `npm run build` in `frontend/`, then creates a Python virtual environment in `backend/`, installs `backend/requirements.txt`, and starts the server with `uvicorn main:app --host 0.0.0.0 --port 7860`.

To run the parts separately:

```bash
# Frontend (development)
cd frontend
npm install
npm run dev

# Backend
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 7860
```

## Project structure

```
start.sh              # Builds the frontend, then starts the FastAPI backend on port 7860
project.md            # Placeholder project notes file
frontend/             # React + Vite + Tailwind frontend
  package.json        # React 19, Vite 5, Tailwind CSS, Framer Motion
  vite.config.js
  tailwind.config.js
  postcss.config.js
  eslint.config.js
  netlify.toml        # Netlify deployment configuration
  src/                # Frontend source
  public/             # Public files
backend/              # FastAPI backend
  main.py             # API: analyze by URL, analyze by upload, chat, health; serves frontend and media
  ai_service.py       # Whisper transcription/metrics and Gemini transcript chat
  requirements.txt
  Dockerfile
  COOKIES_GUIDE.md    # Cookie guidance for downloads
  media/              # Processed videos served for playback
_archive/             # Archived files
```
