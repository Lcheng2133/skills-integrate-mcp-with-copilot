<!-- .github/copilot-instructions.md - Guidance for AI coding agents -->
# Copilot / AI agent instructions for this repository

Summary
- Small FastAPI demo that serves a static front-end and an in-memory activities API.
- Key files: `src/app.py` (FastAPI app), `src/README.md` (developer notes), `src/static/*` (UI), `.vscode/mcp.json` (MCP integration endpoint).

What the project does (big picture)
- Serves a single-page static UI at `/static/index.html` and exposes a small REST API under `/activities`.
- Data is stored in-memory in `src/app.py` as a dict keyed by activity name. No persistence layer.

Quick developer contract (inputs / outputs / success)
- Run server: use Uvicorn (see example below). The application exposes:
  - GET `/activities` -> JSON object mapping activity name -> activity info
  - POST `/activities/{activity_name}/signup?email=...` -> 200 + message or 4xx error
  - DELETE `/activities/{activity_name}/unregister?email=...` -> 200 + message or 4xx error
- Errors use FastAPI HTTPException with appropriate 4xx codes (404 for not found, 400 for bad requests).

How to run locally (concrete)
1. Install dependencies (repo includes `requirements.txt`):

   pip install -r requirements.txt

2. Start with Uvicorn (recommended):

   # from repository root
   uvicorn src.app:app --reload --host 0.0.0.0 --port 8000

Notes: README suggests `python app.py`, but `src/app.py` only defines the FastAPI `app` object and does not call `uvicorn.run`. Use the uvicorn command above.

Useful examples (exact snippets an agent can use)
- List activities (curl):

  curl -s http://localhost:8000/activities | jq .

- Sign up a student:

  curl -X POST "http://localhost:8000/activities/Chess%20Club/signup?email=student@mergington.edu"

Key code patterns & project-specific conventions
- Activities are keyed by their human-readable name (case-sensitive) in `activities` dict inside `src/app.py`.
- Student identity is an email string; signup/unregister use an `email` query param rather than a JSON body.
- Static files are mounted with `app.mount('/static', StaticFiles(directory=...))` and the root redirects to `/static/index.html` in `src/app.py`.

Important edge-cases to handle when changing code
- Activity names are used as dict keys; renaming or normalizing names must consider current keys in memory.
- Concurrency: current storage is not thread-safe for concurrent modifications. Avoid adding background workers that mutate `activities` without synchronization if keeping in-memory storage.
- No persistence: server restart clears all signups — make this explicit in changes that assume durability.

Integration points & external dependencies
- No external databases. Dependencies are the typical FastAPI stack (FastAPI, Uvicorn). See `requirements.txt`.
- `.vscode/mcp.json` exists and points to an MCP server; do not remove when modifying workspace-level settings used by automation.

Where to look for examples and edit points
- `src/app.py` — primary implementation and routes.
- `src/README.md` — quick project notes and original run instructions (contains the outdated `python app.py` guidance).
- `src/static/index.html`, `src/static/app.js` — front-end that calls the API; useful to inspect for expected request shapes.

If you modify APIs
- Update `src/README.md` and adjust the example curl snippets here.
- Keep responses consistent (shape and HTTP codes). Many automated tests or exercises may expect the current shapes.

When in doubt (questions to ask the repo owner)
- Should activities be treated case-insensitively?
- Is there an intended persistence/DB migration planned?

End of file
