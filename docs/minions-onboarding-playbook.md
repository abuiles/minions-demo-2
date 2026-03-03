# Minions Game: Onboarding & Operator Playbook

This guide captures the API workflow, backlog handling, and the operator demo checklist for the *Minions Game* experience. It is intended for engineers who need to integrate with the game APIs and for Ops/onboarding leads who guide new players through the operator demo.

## 1. API Workflow

### 1.1 Overview and Sequence
1. **Create or resume a gameplay session** — `POST /api/v1/games`
   - Triggered from the onboarding portal when a new team loads the demo. If the team already has a session, the portal falls back to `GET /api/v1/games?team=<team-id>` and picks the latest `ACTIVE` state.
2. **Warm-up players & assign loadout** — `POST /api/v1/games/{game_id}/players`
   - Adds the initial pilot/minion roster and maps their loadouts to the current mission profile.
3. **Submit mission plan** — `POST /api/v1/games/{game_id}/missions`
   - Accepts the operator’s choices: path selection, energy packages, optional narrative hooks.
4. **Stream state updates** — `GET /api/v1/games/{game_id}/state` (polling/webhooks)
   - Used by dashboards to show score, active hazards, and player readiness.
5. **Trigger operator action** — `POST /api/v1/games/{game_id}/actions`
   - Examples include `RESUPPLY`, `ABORT`, `DEPLOY_GADGET`. The backend validates payloads and returns the new `state` summary in the response.
6. **Close session and harvest analytics** — `POST /api/v1/games/{game_id}/complete`
   - Used post-demo to capture metrics, export logs, and mark the session as archived.

### 1.2 Payload Examples
| Endpoint | Sample JSON | Notes |
| --- | --- | --- |
| `POST /api/v1/games` | `{ "team_id": "team-101", "mode": "tutorial", "difficulty": "medium" }` | Returns `game_id`, `created_at`, and seeded hazards.
| `POST /api/v1/games/{game_id}/players` | `{ "players": [{"id": "agent-wb", "loadout": "jetpack", "role": "pilot"}] }` | Validate loadout exists before sending.
| `POST /api/v1/games/{game_id}/missions` | `{ "mission_plan": [{"route": "north_pass", "focus": "resource", "priority": 1}] }` | `priority` determines scheduling order for composite objectives.
| `POST /api/v1/games/{game_id}/actions` | `{ "action": "RESUPPLY", "payload": {"resource": "energy", "amount": 250} }` | Action response includes `new_state` and `events` array.
| `POST /api/v1/games/{game_id}/complete` | `{ "feedback": "Demo concluded", "metrics": {"score": 8900} }` | System returns `postgame_report_url` if analytics upload succeeded.

Include these steps in onboarding docs and share the payload templates with the integration team so they can adapt the sample UI hooks.

## 2. Backlog Usage & Flow

1. **Backlog board structure**:
   - *New* → *Sizing* → *In Progress* → *Ready for Ops* → *Done*.
   - Tickets should be tagged with `game-api`, `demo-flow`, or `operator-ops` depending on the focus.
2. **Work intake**:
   - Use the `Operator Demo` epic for stories that change the demo flow or add new API hooks.
   - For urgent fixes (e.g., API endpoint regression during a live demo), open a `bug` with `Priority: Live Demo` so it bypasses the sizing lane.
3. **Backlog hygiene**:
   - During triage, confirm each ticket links to the API contract (schema PR or OpenAPI file). If not, ping the SDK owner for the missing definition.
   - Maintain checklist per ticket: `API contract validated`, `Playbook updated`, `QA did dry run` (checkbox fields on backlog card).
4. **Operator-ready stories**:
   - Once a ticket reaches *Ready for Ops*, assign it to a demo operator and mark steps:
     1. Confirm the API endpoint is deployed to the staging slot.
     2. Attach the `Demo Checklist` update (see Section 3).
     3. Schedule a validation run (Section 4). 

Use backlog dashboards to filter `operator-demo` + `status:notclosed` for upcoming sessions, so triage can prep the right environment well ahead of time.

## 3. Operator Demo Flow & Checklist

### 3.1 Walkthrough Flow
1. **Pre-demo readiness (30 min before)**
   - Verify staging environment is running and the latest build is deployed.
   - Run `GET /api/v1/health` to confirm all services are `status: ok`. If any service is `degraded`, consult the incident playbook.
2. **Kickoff with team**
   - Introduce the game narrative and highlight what APIs they will spot in the logs.
   - Reiterate constraints (max 4 players per demo, 2-minute decision windows).
3. **Live operation**
   - Launch the session with `POST /api/v1/games` and walk through log streams from `GET /api/v1/games/{game_id}/state`.
   - When players submit their mission plan, show how operator actions (Section 1) influence the hazard visualization.
4. **Post-demo debrief**
   - Complete `POST /api/v1/games/{game_id}/complete` while narrating how metrics get captured.
   - Share dashboards showing score and telemetry (logs, stream events, backlog ticket if any). 

### 3.2 Demo Checklist (tick boxes for operators)
- [ ] Environment verified (`GET /api/v1/health`).
- [ ] API payload templates pre-filled (game creation, players, missions, actions).
- [ ] Hazards/resupply scenarios rehearsed (at least two action combos).
- [ ] Demo script aligned with backlog ticket (if a new feature is featured).
- [ ] Post-demo metrics export tested (confirm `postgame_report_url`).
- [ ] Troubleshooting notes ready (Section 4 references). 

Operators should run this checklist every time they launch a production or staged demo to ensure predictable behavior and to avoid last-minute surprises.

## 4. Troubleshooting & Validation Guidance

1. **Common failure modes**:
   - *Game creation fails* (`422`/`500`): Inspect backlog ticket to see if schema changed; double-check that `team_id` and `mode` are valid per new drop-down options.
   - *Player loadouts not recognized*: Ensure new loadouts are registered in `POST /api/v1/loadouts` (maintained by the gameplay team) before shipping the change.
   - *Action queue timeouts*: If `POST /api/v1/games/{game_id}/actions` is returning `429` or `timeout`, the backend may be overloaded. Retry with exponential backoff and flag the issue in the `operator-demo` backlog for capacity improvements.
2. **Validation routine**:
   - After the doc changes land, run the mock script (if available) or a manual session through the API workflow from Section 1. Capture JSON responses at each step and log them with timestamps.
   - Cross-reference logs from `GET /api/v1/games/{game_id}/state` with the payloads to ensure events correlate.
   - Confirm the backlog entry is marked `Verified` and the operator demo checklist is attached.
3. **Operational tools**:
   - Use the in-house log viewer to inspect `game.events` stream; filtering by `game_id` isolates the live session.
   - If telemetry shows `missing_action` events, rerun the mission plan with manual adjustment and re-issue `POST /api/v1/games/{game_id}/actions` to confirm recovery.
4. **Validation sign-off**:
   - Once the session runs cleanly, note the validation timestamp on the backlog card, then transition the card to *Done*.
   - Communicate any residual risks (e.g., `action_queue` still congested) via the `operator-demo` Slack channel so the on-call team can follow up.

Keep this playbook synchronized with the backlog so every API change and demo upgrade is traceable and easy to reproduce.
