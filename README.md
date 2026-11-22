# CampusTreasury AI

CampusTreasury AI is a React frontend connected to a FastAPI backend with SQLite persistence, event and document management, organisation settings, and a help assistant.

## Windows 11 setup

Open the project folder in VS Code, then open two VS Code terminals.

### Terminal 1 - Backend

```powershell
cd backend
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
copy .env.example .env
uvicorn app.main:app --reload
```

If PowerShell blocks activation, use Command Prompt or run:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

### Terminal 2 - Frontend

```powershell
cd frontend
copy .env.example .env
npm.cmd install
npm.cmd run dev
```

Open the URL printed by Vite, usually http://127.0.0.1:5173. Vite proxies `/api` requests to the backend at http://127.0.0.1:8000.

## Run as one application

Build the frontend, then run FastAPI from the backend directory:

```powershell
cd frontend
npm.cmd run build
cd ..\backend
.\.venv\Scripts\python.exe -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Open http://127.0.0.1:8000. FastAPI serves the built website and its API from the same origin. API documentation remains available at http://127.0.0.1:8000/docs.

## Deploy frontend to Vercel

The repository includes a root `vercel.json` for building and hosting the React
frontend. The FastAPI backend, SQLite database, and uploaded files need a
persistent backend host; `render.yaml` configures the backend for Render with a
persistent disk.

1. Push this project to a Git provider and import it into Vercel. Keep the
   project root directory as `.`; Vercel reads the build commands and output
   directory from `vercel.json`.
2. Create the backend on Render using the repository's `render.yaml` Blueprint.
   The configured persistent disk requires a paid Render web-service plan.
3. In Vercel project settings, add `VITE_API_URL` with the deployed Render
   backend URL, for example `https://campustreasury-api.onrender.com`, then
   redeploy the frontend.
4. In Render's backend environment settings, replace the example
   `CORS_ORIGINS` value with the exact Vercel site origin, such as
   `https://your-project.vercel.app` (no path or trailing slash). Include
   additional preview or custom-domain origins as comma-separated values, then
   restart the backend.
5. Optionally add `GEMINI_API_KEY` to the Render backend environment to enable
   open-ended AI responses. Never add it to Vercel or frontend environment
   variables.

The backend's SQLite database and uploaded documents are stored on Render's
persistent disk. Keep that disk attached to the service; deleting it permanently
deletes that data. The local development `.env` files are not used by Vercel or
Render.

## Events, documents, settings, and help assistant

The workspace navigation now opens functional Events, Documents, and Settings pages. Event and organisation changes are saved in SQLite. Uploaded PDF/JPG/PNG files are held under `backend/uploads` and are served only through the document API.

The help assistant answers common CampusTreasury questions in offline quick-help mode by default. To enable open-ended Gemini answers, create a Gemini API key in Google AI Studio, then add it to `backend/.env`:

```env
GEMINI_API_KEY=your_real_key_here
GEMINI_MODEL=gemini-2.5-flash
```

Keep the key only in the backend environment file; never put it in `frontend/.env`. Restart the backend after adding it. Without a key, the assistant remains usable for common product questions and clearly indicates that it is using offline quick help.

The Gemini assistant is a support guide and does not read private documents or make audit decisions. Its answers are informational; audit findings still need human review.
