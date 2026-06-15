# Render Integration & Deployment Plan — dnd-merchants-generator

## Context

The repo is a bare Django 6.0 scaffold (no domain app yet) with a GitHub remote already
wired (`github.com/MateuszKTF/brave_kurs.git`, branch `master`). `context/foundation/infrastructure.md`
selected **Render** as the deploy platform. This plan operationalizes that decision into concrete
config files and a deploy runbook.

**Decisions confirmed with the user (override two points in `infrastructure.md`):**
- **Database = managed Render Postgres** (not SQLite-on-disk). This removes the persistent-disk
  requirement and its build-time pitfalls.
- **Tier = free-tier prototype first**, then a deliberate upgrade to paid before any real campaign
  data is entered.

**Live facts verified during planning (2026-06-15):**
- ✅ **Python 3.14 is now the Render default** (3.14.3, since 2026-02-11) — the "3.14 unverified" risk
  in `infrastructure.md` is **resolved**. We target 3.14.
- ✅ **Free web services have an ephemeral filesystem and cannot attach a persistent disk** — which is
  exactly why we use Postgres for durability, not local SQLite.
- ⚠️ **Persistent disks are NOT mounted during build/pre-deploy.** Not a problem for us (Postgres is
  network-attached), but it means `migrate` against Postgres is *safe* to run in `build.sh`.
- ⚠️ **Free Postgres self-deletes 30 days after creation, with no backups.** This is the headline
  risk of the chosen path and gets explicit mitigation in Phase 5.

**Intended outcome:** a reproducible, env-var-driven Django app that builds and runs on Render's free
tier from a `render.yaml` blueprint, with a documented upgrade path to a durable paid configuration.

---

## Phase 1 — Local dependency hygiene
- [ ] Create a real virtualenv so we don't pollute global Python (per `CLAUDE.md`):
      `py -m venv .venv` → activate → `py -m pip install django==6.0.6 gunicorn whitenoise dj-database-url "psycopg[binary]"`
- [ ] Freeze pins: `py -m pip freeze > requirements.txt`. Confirm it pins at minimum
      `Django==6.0.6`, `gunicorn`, `whitenoise`, `dj-database-url`, `psycopg[binary]`.
      - *Note:* use **`psycopg[binary]`** (psycopg 3). Django 6.0 supports it natively; avoid the
        legacy `psycopg2-binary` unless a build error forces the fallback.
- [ ] Create **`.python-version`** at repo root with `3.14`.

## Phase 2 — Create deploy files (repo root)
- [ ] **`.gitignore`** (currently missing — required before pushing to GitHub):
      ignore `.venv/`, `__pycache__/`, `*.pyc`, `db.sqlite3`, `/staticfiles/`, `.env`.
- [ ] **`build.sh`** (mark executable — `git update-index --chmod=+x build.sh` on Windows since
      `chmod` isn't native):
      ```sh
      #!/usr/bin/env bash
      set -o errexit
      pip install -r requirements.txt
      python manage.py collectstatic --no-input
      python manage.py migrate
      ```
      `migrate` belongs here (Postgres is reachable at build time). It would NOT be safe here for
      SQLite-on-disk — see Context.
- [ ] **`render.yaml`** blueprint at repo root:
      ```yaml
      databases:
        - name: dnd-merchants-db
          plan: free            # 30-day expiry — upgrade before real data (Phase 5)
          region: frankfurt     # co-locate with web service (EU)

      services:
        - type: web
          name: dnd-merchants-generator
          runtime: python
          plan: free
          region: frankfurt
          branch: master
          buildCommand: "./build.sh"
          startCommand: "gunicorn dnd_merchants_generator.wsgi:application"
          envVars:
            - key: DJANGO_SECRET_KEY
              generateValue: true
            - key: DEBUG
              value: "False"
            - key: PYTHON_VERSION
              value: "3.14.3"
            - key: DATABASE_URL
              fromDatabase:
                name: dnd-merchants-db
                property: connectionString
      ```
      - Using **WSGI** (`wsgi:application`) per `infrastructure.md` — the app is plain CRUD + a price
        recompute, no async need. (ASGI+uvicorn is the alternative if async is ever added.)
      - `region: frankfurt` chosen for an EU user; web + DB co-located to minimize latency. Adjust if
        a different region is preferred.

## Phase 3 — Production-harden `dnd_merchants_generator/settings.py`
Edit the scaffold settings (currently `DEBUG=True`, hardcoded `SECRET_KEY`, `ALLOWED_HOSTS=[]`):
- [ ] `import os`, `import dj_database_url` at top.
- [ ] `SECRET_KEY = os.environ.get("DJANGO_SECRET_KEY", "<existing-insecure-default-for-local>")`
- [ ] `DEBUG = os.environ.get("DEBUG", "True") == "True"`
- [ ] **ALLOWED_HOSTS / CSRF** driven by Render's auto-injected hostname:
      ```python
      ALLOWED_HOSTS = ["localhost", "127.0.0.1"]
      RENDER_HOST = os.environ.get("RENDER_EXTERNAL_HOSTNAME")
      if RENDER_HOST:
          ALLOWED_HOSTS.append(RENDER_HOST)
          CSRF_TRUSTED_ORIGINS = [f"https://{RENDER_HOST}"]
      ```
      *Edge case:* without `CSRF_TRUSTED_ORIGINS`, admin/login POSTs over HTTPS return **403** on the
      `.onrender.com` domain. This line prevents that.
- [ ] **WhiteNoise**: add `"whitenoise.middleware.WhiteNoiseMiddleware"` immediately **after**
      `SecurityMiddleware` in `MIDDLEWARE`.
- [ ] **Static files**:
      ```python
      STATIC_ROOT = BASE_DIR / "staticfiles"
      STORAGES = {
          "default": {"BACKEND": "django.core.files.storage.FileSystemStorage"},
          "staticfiles": {"BACKEND": "whitenoise.storage.CompressedManifestStaticFilesStorage"},
      }
      ```
- [ ] **Database with safe local fallback** (so local dev still uses SQLite when `DATABASE_URL` is
      unset, and Render uses Postgres):
      ```python
      DATABASES = {
          "default": dj_database_url.config(
              default=f"sqlite:///{BASE_DIR / 'db.sqlite3'}",
              conn_max_age=600,
              ssl_require=bool(os.environ.get("RENDER_EXTERNAL_HOSTNAME")),
          )
      }
      ```

## Phase 4 — Deploy to Render (free prototype)
- [ ] **Local smoke test first** (catch errors before the cloud round-trip):
      `py manage.py collectstatic --no-input` and `py manage.py migrate` and `py manage.py runserver`
      against local SQLite, with `DEBUG=False` exported to exercise the prod path.
- [ ] **Commit & push** all new files to `master`: `git add . && git commit && git push origin master`.
- [ ] **[HUMAN GATE — dashboard]** Render dashboard → **New → Blueprint** → connect
      `MateuszKTF/brave_kurs` → **Apply**. Creates the free web service + free Postgres from
      `render.yaml`. (First-time blueprint apply is dashboard-driven; account creation is human-only.)
- [ ] Render runs `build.sh` (installs, collectstatic, migrate against Postgres) then starts gunicorn.
- [ ] **Verify**: open the `.onrender.com` URL (expect Django welcome page since no app/views yet);
      hit `/admin/` to confirm static files + CSRF + DB work. Create a superuser if needed via
      `render ssh <service> -- python manage.py createsuperuser` (or dashboard Shell).
- [ ] **Logs/ops** (agent-runnable, unattended): `render logs --resources <service-id> --tail`, or the
      GA Render MCP server (`https://mcp.render.com/mcp`). `render login` is an interactive human step
      before CLI use.

## Phase 5 — Pre-production upgrade (before ANY real campaign data)
The free path is a prototype only. Schedule this before data matters:
- [ ] **Calendar reminder for the 30-day free-Postgres expiry** (created 2026-06-15 → hard deadline
      ~2026-07-15). Free Postgres deletion is **silent, permanent, and unbacked**.
- [ ] **Upgrade web service → Starter (~$7/mo)** in the dashboard to remove ~60s cold starts (protects
      the PRD <15s session-tempo goal). [HUMAN-only: billing change.]
- [ ] **Move to a paid Postgres tier with backups.** *Verify the upgrade mechanism* — a free→paid
      Postgres upgrade may require provisioning a new paid instance and migrating via
      `pg_dump`/`pg_restore` rather than an in-place plan change. Take a `pg_dump` backup **before**
      touching the instance regardless. [HUMAN-only: DB resize/delete.]
- [ ] Re-point `DATABASE_URL` (via `fromDatabase`) if the DB instance changed, redeploy, confirm data.

---

## Files created / modified
| Path | Action |
|---|---|
| `requirements.txt` | **create** — pinned deps |
| `.python-version` | **create** — `3.14` |
| `.gitignore` | **create** — venv/sqlite/staticfiles/env |
| `build.sh` | **create** — install + collectstatic + migrate |
| `render.yaml` | **create** — free web + free Postgres blueprint |
| `dnd_merchants_generator/settings.py` | **modify** — env-driven secret/debug/hosts, WhiteNoise, STATIC_ROOT, dj-database-url |

`wsgi.py` needs no change — `dnd_merchants_generator.wsgi:application` already matches the start command.

## Verification (end-to-end)
1. **Local**: `.venv` active, `DEBUG=False`, `py manage.py collectstatic --no-input && py manage.py migrate && py manage.py runserver` → site loads, `/admin/` static assets render.
2. **Build parity**: `bash build.sh` locally (Git Bash) completes without error.
3. **Cloud**: Blueprint applies; Render build log shows successful `collectstatic` + `migrate`; service reaches "live".
4. **Smoke**: `.onrender.com` root + `/admin/` login (no 403 = CSRF origins correct; styled = WhiteNoise correct; login persists = Postgres correct).
5. **Logs clean**: `render logs --tail` shows no `DisallowedHost`, `ProgrammingError`, or static-manifest errors.

## Edge cases & extra support
- **Free Postgres 30-day expiry** → Phase 5 reminder + `pg_dump` before upgrade.
- **CSRF 403 on admin over HTTPS** → `CSRF_TRUSTED_ORIGINS` set from `RENDER_EXTERNAL_HOSTNAME` (Phase 3).
- **Local dev breakage** → `dj_database_url.config(default=sqlite...)` keeps local SQLite when `DATABASE_URL` is unset.
- **`psycopg[binary]` build failure** → fallback to `psycopg2-binary` and re-freeze.
- **uv vs pip** → Render now defaults to `uv`, but our explicit `./build.sh` uses `pip install -r requirements.txt`, so behavior is deterministic regardless of Render's default.
- **Cold start ≠ re-migrate** → on free tier, spin-up does NOT re-run `build.sh`; `migrate` only runs on deploy. No surprise schema churn on wake.
- **Disk-not-at-build pitfall** → avoided by choosing Postgres; documented so a future SQLite revisit doesn't reintroduce it.

## Out of scope (per infrastructure.md)
Dockerfiles, GitHub Actions CI/CD, multi-region/HA. Domain app code (models/views) is separate product work — this plan only makes the scaffold deployable.
