# dbx-hello-app

A single-file Databricks App (`app.py`) — the "Tallent Management Portal". It serves an HTML UI over a stdlib `http.server` and persists an employee training register in a Unity Catalog Delta table (`workspace.default.tallent_training_register`) via `databricks-sql-connector`.

## Cursor Cloud specific instructions

### Services
There is one service: the `app.py` HTTP server.

- Run it with the venv interpreter: `.venv/bin/python app.py`
- It listens on `0.0.0.0:$DATABRICKS_APP_PORT` (default `8000`).
- Python deps live in `.venv` (created from `requirements.txt`). The update script keeps this venv in sync.

### Databricks credentials (required for the data layer)
The web server starts and returns HTTP 200 **without** any Databricks credentials, but every page render attempts a SQL connection. Without credentials the dashboard/table is replaced by a "Catalog SQL connection error" banner, and `POST /add-employee` redirects back with an error. This is expected, not a code bug.

To exercise the real data functionality (seed/view the register, add employees), the process needs Databricks unified auth in its environment:
- `DATABRICKS_HOST` (workspace URL)
- An auth method recognized by `databricks.sdk.core.Config` (e.g. `DATABRICKS_TOKEN`, or a `~/.databrickscfg` profile, or OAuth client id/secret)
- `DATABRICKS_WAREHOUSE_ID` (a SQL warehouse the user can `CAN_USE`; `app.py` falls back to a hardcoded default if unset)

The optional AI copilot helpers (`ask_deepseek` / `ask_openai_compatible`) read `DATABRICKS_AI_ENDPOINT` or `OPENAI_COMPAT_*` / `AI_BACKEND`, but those code paths are not wired to any HTTP route.

### Routing quirk
`do_GET` renders the portal page for **any** path (there is no GET 404), so unknown GET paths still return HTTP 200 with the full page. Only `POST` distinguishes routes (`/add-employee`).

### Tests / lint
There is no test suite and no lint configuration in this repo. Use `.venv/bin/python -m py_compile app.py` as a basic syntax check.
