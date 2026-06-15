---
bootstrapped_at: 2026-06-15T10:03:54Z
starter_id: django
starter_name: Django
project_name: dnd-merchants-generator
language_family: python
package_manager: uv
cwd_strategy: native-cwd
bootstrapper_confidence: verified
phase_3_status: ok
audit_command: pip-audit
---

## Hand-off

Verbatim copy of `context/foundation/tech-stack.md`:

```yaml
starter_id: django
package_manager: uv
project_name: dnd-merchants-generator
hints:
  language_family: python
  team_size: solo
  deployment_target: fly
  ci_provider: github-actions
  ci_default_flow: auto-deploy-on-merge
  bootstrapper_confidence: verified
  path_taken: standard
  quality_override: false
  self_check_answers: null
  has_auth: true
  has_payments: false
  has_realtime: false
  has_ai: false
  has_background_jobs: false
```

> ## Why this stack
>
> A solo Mistrz Gry building a DnD merchants generator as a small web app on a hard
> 3-week deadline needs the boring-but-required half — email/password auth, per-user
> data isolation, durable persistence, and CRUD over saved shops — to come for free.
> Django is the recommended default for `(web, python)` and ships exactly that:
> ORM, migrations, auth, and admin on day 1, leaving the calendar for the two real
> domain rules (matched-pool assortment draw and the Persuasion price recompute),
> which are plain Python on top. It clears three of four agent-friendly gates
> outright; the typed gate is the Python-family caveat (Django is duck-typed, not
> statically typed), but its convention discipline, large training corpus, and
> current docs more than compensate, and bootstrapper confidence is verified, so
> scaffolding will be smooth. Auth and persistence feature flags are set; payments,
> realtime, AI, and background jobs are out of scope per the PRD non-goals.
> Deployment targets Fly.io (the Django card default) with GitHub Actions running
> auto-deploy on merge — the standard shape for a solo project.

## Pre-scaffold verification

| Signal       | Value                                          | Severity | Notes                                                              |
| ------------ | ---------------------------------------------- | -------- | ------------------------------------------------------------------ |
| npm package  | not run                                        | n/a      | Django's cmd_template invokes `django-admin`, not an npm `create-*` CLI |
| GitHub repo  | not run                                        | n/a      | card `docs_url` is `https://docs.djangoproject.com` — not a GitHub repo URL |

No recency signal was available for this starter. Per the slot's WARN-AND-CONTINUE policy, scaffolding proceeded.

## Scaffold log

**Resolved invocation**: `django-admin startproject dnd_merchants_generator .`
**Strategy**: native-cwd
**Exit code**: 0
**Pre-flight files-to-touch**: `manage.py`, `dnd_merchants_generator/` (project package)
**Files written by CLI**: 6 — `manage.py`, `dnd_merchants_generator/__init__.py`, `dnd_merchants_generator/settings.py`, `dnd_merchants_generator/urls.py`, `dnd_merchants_generator/asgi.py`, `dnd_merchants_generator/wsgi.py`
**Pre-existing files preserved**: `CLAUDE.md`, `idea-notes.md`, `context/`, `.claude/` (none overwritten; no `.scaffold` siblings produced)
**Substitution note** — The card's `cmd_template` is `django-admin startproject {name} .`, where the trailing `.` already directs the scaffold into the current directory (the native-cwd behavior). The literal native-cwd rule (`{name}` → `.`) was tested first and rejected by Django (`CommandError: '.' is not a valid project name`), because in this template `{name}` is the Django *project name* (a Python package identifier), not a directory. The hand-off `project_name` `dnd-merchants-generator` is itself not a valid Python identifier (hyphens), so it was sanitized to `dnd_merchants_generator` and used as the project name; the template's trailing `.` provided the cwd placement. The scaffold then succeeded with exit code 0.

**Toolchain note** — `package_manager: uv` was recorded in the hand-off but is not used by Django's scaffold command (the `cmd_template` carries no `{pm}` placeholder), and `uv` is not installed on this machine. The card's `pre` step (`pip install django`) was run via the available Python launcher (`py -m pip install django`, Django 6.0.6) because the bare `python`/`uv` commands were unavailable. No project virtualenv was created (no `uv` to manage one); Django was installed into the active Python 3.14.3 environment.

**.gitignore handling**: absent in scaffold (Django 6.0.6 `startproject` produced no root `.gitignore`)
**.bootstrap-scaffold cleanup**: not applicable (native-cwd writes directly into cwd; no temp directory used)

## Post-scaffold audit

**Tool**: `pip-audit` (run as `py -m pip_audit --format json`; `pip-audit` 2.10.1 installed for this run)
**Summary**: 0 CRITICAL, 0 HIGH, 0 MODERATE, 0 LOW — **for the project's dependency subtree**
**Direct vs transitive**: not distinguished by this tool

The project's actual dependency subtree was audited and is clean:

| Package  | Version  | Vulnerabilities |
| -------- | -------- | --------------- |
| django   | 6.0.6    | 0               |
| asgiref  | 3.11.1   | 0               |
| sqlparse | 0.5.5    | 0               |
| tzdata   | 2025.3   | 0               |

**Scoping caveat** — Because no project virtualenv exists (`uv` unavailable to create one), `pip-audit` ran against the whole active Python environment, which already contained many packages unrelated to this scaffold. The whole-environment scan reported 24 advisory entries across 8 packages (e.g., `aiohttp`, `urllib3`), but **none of those packages are part of the Django scaffold's dependency tree** — they pre-existed in the shared environment. The table above reflects the audit findings scoped to the four packages the scaffold actually introduced. Recommended next step: create a dedicated project virtualenv (e.g., `py -m venv .venv`, or install `uv` and `uv venv`) so future audits are scoped to project dependencies only.

#### CRITICAL findings

None.

#### HIGH findings

None.

#### MODERATE findings

None (within the project dependency subtree).

#### LOW / INFO findings

None (within the project dependency subtree).

## Hints recorded but not acted on

| Hint                    | Value                  |
| ----------------------- | ---------------------- |
| bootstrapper_confidence | verified               |
| quality_override        | false                  |
| path_taken              | standard               |
| self_check_answers      | null                   |
| team_size               | solo                   |
| deployment_target       | fly                    |
| ci_provider             | github-actions         |
| ci_default_flow         | auto-deploy-on-merge   |
| has_auth                | true                   |
| has_payments            | false                  |
| has_realtime            | false                  |
| has_ai                  | false                  |
| has_background_jobs     | false                  |

These hints were read from the hand-off and recorded here for audit-trail completeness. v1 of the bootstrapper surfaces but does not act on them — no CI/CD scaffolding, no auth wiring, no deployment configuration. A future "Memory Architecture" skill will act on them.

## Next steps

Next: a future skill will set up agent context (CLAUDE.md, AGENTS.md). For now, your project is scaffolded and verified — happy hacking.

Useful manual steps in the meantime:
- `git init` (if you have not already) to start your own repo history.
- Create a project virtualenv so dependencies (and future audits) are isolated from the global Python environment — e.g. `py -m venv .venv` and activate it, or install `uv` (the package manager you picked) with `uv venv` + `uv add django`. Pin dependencies into a `requirements.txt` or `pyproject.toml`.
- Run the initial migrations and create a superuser to exercise the built-in auth/admin you picked Django for: `py manage.py migrate`, then `py manage.py createsuperuser`.
- Review audit findings per your project's risk tolerance — the full breakdown is in this log (the project subtree is clean).
