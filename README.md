# minions-demo-2

A minimal front-end shell for the Minions demo game, built for static deployment on GitHub Pages. All gameplay logic lives inside the browser—no backend is required.

## Repository status

- Frontend-only gameplay logic and rendering in `index.html`
- No backend, API, auth, or worker layer required
- GitHub Pages deployment target: repository root `index.html`

## Local run

1. Install nothing (no dependencies).
2. Serve from repository root over HTTP (file:// loading can fail in some browsers):

   ```sh
   python3 -m http.server 4173
   ```

3. Open:

   - `http://localhost:4173`

4. Validate behavior:
   - Page loads and draws the board.
   - Click **Start**, then **Pause**, then **Reset** to confirm controls exist.
   - Target/score/timer UI updates as you play.

### GitHub Pages previews for PRs

This repository now uses a GitHub Actions workflow to deploy:

- Production: every push to `main` deploys to the repository’s production Pages site.
- Pull Requests: every PR targeting `main` gets a branch-specific PR preview deployment.

On PR update, the workflow posts/updates a comment in the PR thread with the latest preview URL, and the deployment appears in the workflow logs as a verification point.

### Local validation command

Use this command to quickly validate local availability:

```sh
curl -sSf http://localhost:4173 | head -n 1
```

Expected: a successful response with HTML output and no error/timeout.

## GitHub Pages deployment

### Pages playbook

1. Ensure the working tree contains the built artifact (no build step required):
   - `index.html` exists at repository root.
2. In GitHub, open **Settings > Pages**:
   - **Source**: `GitHub Actions`
3. Verify the main workflow is configured and pushed:
   - `.github/workflows/pages-preview.yml` exists.
3. Save and push changes to that branch.
4. The workflow runs on `push` to `main` for production deployment and `pull_request` for preview deployments.
5. Wait for Pages publication to finish, then open:

   - `https://<your-github-username>.github.io/<repository-name>/`

### PR preview verification

- Open the **Checks** tab for the PR run:
  - `Deploy PR preview to Pages` logs include the preview URL.
- Open the PR conversation:
  - the workflow comments with a message containing the latest preview link.

6. Optional hard refresh after deployment to clear stale browser cache.

### Pages URL validation

For an owned deployment URL, run:

```sh
export PAGES_URL="https://<your-github-username>.github.io/minions-demo-2/"
curl -sSI "$PAGES_URL" | head -n 1
```

Expected outcome:

- HTTP status `200 OK`
- Content-Type contains `text/html`

## Deployment edge cases

- If the repo name has different casing or path, update the Pages URL accordingly.
- Confirm Pages branch is the same branch receiving changes (example: `main`).
- If users report seeing old content, perform a hard reload (Ctrl/Cmd + Shift + R).
- Ensure repository is not private with Pages disabled by policy.

## Task handoff checklist

Use this checklist at handoff time:

- [ ] Deployment docs in this README are accurate for the target GitHub org/user/repo.
- [ ] `index.html` remains at repository root for direct static publish.
- [ ] Local playbook and Pages URL validation are documented.
- [ ] No build or backend dependencies required for deployment.
- [ ] Known risks logged in `TASK_HANDOFF.md` (if any).

## Runtime validation targets

Run these in order:

1. `python3 -m http.server 4173`
2. `curl -sSf http://localhost:4173 | head -n 1`
3. `curl -sSI "$PAGES_URL" | head -n 1` (after deployment)
4. Manual UI smoke test in browser (controls + score/timer)

---
