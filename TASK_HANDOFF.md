# minions-demo-2 handoff notes

## Scope completed
- Added static deployment playbook and Pages URL validation guidance to `README.md`.
- Clarified frontend-only runtime and no-backend constraints in docs.
- Added explicit deployment edge cases and local smoke checks.

## Files changed
- `README.md`
- `TASK_HANDOFF.md`

## Deployment command summary
- Local: `python3 -m http.server 4173`
- Local health check: `curl -sSf http://localhost:4173 | head -n 1`
- Pages validation: `curl -sSI "https://<your-github-username>.github.io/<repository-name>/" | head -n 1`

## Preconditions
- `index.html` must remain at repository root.
- GitHub Pages source set to `main` (or selected branch) + `/ (root)`.

## Known risks / follow-up
- Pages publication is async; caches may show stale content briefly.
- If the URL is inaccessible, verify repository visibility and Pages settings.
