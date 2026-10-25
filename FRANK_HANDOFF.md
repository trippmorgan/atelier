# Handoff: wire the Atelier to Frank

Paste everything below the line into Frank (or Claude Code on Voldemort).

---

Build `atelier-api`, the backend for the Atelier, a static web app on GitHub Pages that Marjorie, Ellie, and Tripp use to make art with Frank. The front end is finished. Read `API.md` in this folder for the full contract and match it exactly; `index.html` shows how every call is used (search for `const api =`).

## Isolation comes first

This service will be reachable from phones outside the house, and Voldemort also runs practice systems. So:

1. Run it as its **own service on its own port** (suggest 30200), with its own SQLite database and image folder under the workspace (e.g. `~/atelier-data/`).
2. It must have **no access** to the Surgical Command Center database, PlaudAI, the PHI ledger, Athena or UltraLinq tooling, or any practice credentials. No shared Postgres connection. Different OS user if practical.
3. Expose **only this port** with `tailscale serve --https=443 localhost:30200`. Preferred: Marjorie's and Ellie's phones join the tailnet, so nothing is on the public internet. Tell me before using Funnel or a Cloudflare tunnel instead.
4. Auth: one studio code per person in a config file, as `Authorization: Bearer <code>`. The code decides who someone is; ignore the `by` field in request bodies.
5. CORS: allow `https://<github-user>.github.io` only.

## Frank in the Atelier

- `/api/compose` and `/api/threads/:id/ask` go through Frank's hemispheres, under an **Atelier persona**: an art collaborator talking with teenagers. Keep it warm, honest, and age-appropriate, stay on art and the project, and keep the routes restricted. **No TOOLWEB, no TOOLEXEC**, and no memory of or access to anything outside this service.
- Give the ask endpoint the last ~20 messages of the thread as context, plus the current Lab state, so Frank can talk about training progress accurately.
- Route these through Jev-Omni if that's natural (route: ANSWER / COMPOSE / DECLINE), but don't block on it.

## Image generation

1. Install ComfyUI under the workspace and drive it through its HTTP API. Start with DreamShaper 8 / SD 1.5 for prompt-only, then add SDXL base weights.
2. Map `weights` onto LoRAs when a painter's Lab stage is `ready`. Otherwise fold the painter into the prompt (prompt-only mode) and report `model` honestly.
3. All jobs go through the existing **GPU lease**. Never evict Qwen in the middle of a Telegram exchange. Queue, and put the reason in the job `message` ("Waiting for the GPU…").
4. Keep the **NSFW safety checker on for every output**, plus a prompt filter. A rejected job returns `failed` with a friendly message and saves nothing.
5. Save PNGs under unguessable names (`/files/<uuid>.png`) and serve them without auth.
6. On each new piece, write a provenance entry (prompt hash, seed, weights, model, ideaBy, timestamp) to the mirror-ledger, kept separate from the PHI ledger, and return its id as `ledger`.

## Lab and training windows

- `/api/lab` reports each painter's stage, dataset count, base, and LoRA version from the real training folders.
- `POST /api/lab/windows` creates a `requested` window and notifies Tripp on Telegram. Tripp approves; then Frank schedules it in the lease system, and status moves through `scheduled` → `running` → `done`.
- While a window runs, `/api/status` returns `state: "training"`.

## Seed data

Create threads `general` ("Studio chat"), `next` ("Next steps"), and `question` ("The big question: who's the artist?"). Port the starting tasks from `seedDb()` in `index.html`.

## Done means

1. `curl` with a valid code returns `/api/status`; a bad code gets 401.
2. In the Atelier: Settings → Frank's address + studio code → **Test connection** says connected.
3. A two-piece generation from a phone off home Wi-Fi lands in the Gallery with a ledger id.
4. Asking Frank in "Next steps" gets a real reply.
5. Confirm by port scan that nothing on Voldemort except this service is reachable through the tailnet share.

Report back with the address, how codes were issued, and anything that differs from `API.md`.
