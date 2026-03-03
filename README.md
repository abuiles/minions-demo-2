# minions-demo-2

A minimal front-end shell for the Minions demo game, built for static deployment on GitHub Pages. All gameplay logic lives inside the browser—no backend is required.

## Local preview
1. Serve the repository root over HTTP (browsers block local file access). For example:
   ```sh
   python3 -m http.server 4173
   ```
2. Open `http://localhost:4173` in your browser and interact with the target, score, and timer controls.
3. The buttons allow starting, pausing, and resetting the session; the target only responds while the timer is running.

## GitHub Pages deployment
1. GitHub Pages works directly from this repository root because `index.html` lives in the default location.
2. In the repository's **Settings > Pages** panel, choose the `main` branch (or whichever branch you are using) and the `/ (root)` folder as the source.
3. Once saved, GitHub Pages will host the demo at `https://<your-username>.github.io/minions-demo-2/`. Refresh the page after pushing updates so the browser loads the latest shell.
4. No build steps or frameworks are necessary, so committing and pushing the static files is enough to update the live demo.

## Local persistence and deterministic scenario runner
- High score and last-run summary are stored in `localStorage` under `minions-demo-2-state-v2`.
- `window.runScenario({...})` executes a deterministic replay using an optional `seed`, `startState`, and `moves` list.
- Local storage parsing is defensive: invalid/corrupt data falls back to defaults instead of breaking gameplay.
