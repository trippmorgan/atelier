# Atelier API contract

The front end (`index.html`) talks to Frank through one adapter object (`api` in the script). In demo mode every call goes to a local mock. Once **Frank's address** is set in Settings, the same calls become the REST requests below.

## Basics

- **Base URL:** whatever is entered in Settings, e.g. `https://voldemort.<tailnet>.ts.net`. It must be **HTTPS**, because GitHub Pages is HTTPS and browsers block mixed content.
- **Auth:** `Authorization: Bearer <studio code>`. Give each person their own code and map code → person server-side. Trust the code, not the `by` field in request bodies.
- **CORS:** allow origin `https://<github-user>.github.io`, methods `GET, POST, PUT, OPTIONS`, and headers `Authorization, Content-Type`.
- **Errors:** non-2xx with `{ "error": "human-readable message" }`. The UI shows that message as a toast.
- **Images:** `url` fields must be loadable by a plain `<img>` tag, which can't send the bearer header. Use unguessable paths (`/files/<random-uuid>.png`) or short-lived signed URLs. A relative URL starting with `/` is prefixed with the base URL.
- **People ids:** `marjorie`, `ellie`, `tripp`, and `frank` for Frank.
- **Painter ids:** `vangogh`, `leonardo`, `bosch`. Weights run from 0 to 1.

## Objects

```jsonc
// Art
{ "id": "a1", "kind": "generated" | "hand" | "reference",
  "title": "Anatomy of Chaos", "ideaBy": "marjorie", "by": "frank",
  "idea": "a heart drawn like a map…",          // optional, generated only
  "prompt": "…", "weights": { "vangogh": 0, "leonardo": 0.85, "bosch": 0.6 },
  "seed": 4127, "model": "SDXL + vangogh-v1 LoRA",
  "url": "/files/9f1c….png", "note": "", "basedOn": "a0" | null,
  "favs": ["ellie"], "ledger": "sha256:…" | null, "created": "2026-10-24T22:40:00Z" }

// Job
{ "id": "j1", "status": "queued" | "running" | "done" | "failed",
  "progress": 0.4, "message": "Waiting for GPU lease…", "art": [Art] }

// Message
{ "id": "m1", "by": "frank", "text": "…", "at": "ISO time" }

// Task
{ "id": "t1", "title": "…", "owner": "ellie", "status": "todo" | "doing" | "done" }

// Lab
{ "artists": { "vangogh": { "stage": "baseline" | "dataset" | "training" | "ready",
                             "images": 120, "target": 150, "base": "SDXL",
                             "version": "v1" | null, "note": "…" }, … },
  "windows": [ { "id": "w1", "artist": "vangogh", "base": "SDXL", "when": "Tonight after midnight",
                 "note": "", "by": "marjorie", "status": "requested" | "scheduled" | "running" | "done" | "declined",
                 "at": "ISO time" } ] }
```

## Endpoints

| Method | Path | Body | Returns |
|---|---|---|---|
| GET | `/api/status` | | `{ state: "online" \| "training" \| "offline", gpu, nextWindow }` |
| GET | `/api/art` | | `Art[]`, newest first |
| GET | `/api/art/:id` | | `Art` |
| POST | `/api/art` | `{ dataUrl, title, kind, basedOn, note, by }` | `Art` (upload; `dataUrl` is a base64 JPEG ≤ 2048px) |
| POST | `/api/art/:id/favorite` | `{ by, on }` | `Art` |
| GET | `/api/art/:id/comments` | | `Message[]` |
| POST | `/api/art/:id/comments` | `{ by, text }` | any |
| POST | `/api/compose` | `{ idea, weights, by }` | `{ prompt }`, written by Frank |
| POST | `/api/generate` | `{ title, idea, prompt, weights, count (1–4), aspect: square\|portrait\|landscape, seed \| null, ideaBy }` | `Job` |
| GET | `/api/jobs/:id` | | `Job` (UI polls every 2.5s) |
| GET | `/api/threads` | | `[{ id, title }]`. Keep ids `general`, `next`, `question` |
| POST | `/api/threads` | `{ title, by }` | `{ id, title }` |
| GET | `/api/threads/:id/messages` | | `Message[]`, oldest first |
| POST | `/api/threads/:id/messages` | `{ by, text }` | any |
| POST | `/api/threads/:id/ask` | `{ by, text }` | `Message`. Frank's reply, also saved to the thread |
| GET | `/api/tasks` | | `Task[]` |
| POST | `/api/tasks` | `Task` without id | `Task` |
| PUT | `/api/tasks/:id` | partial `Task` | `Task` |
| GET | `/api/lab` | | `Lab` |
| POST | `/api/lab/windows` | `{ artist, base, when, note, by }` | window object. Lands as `requested`, and Tripp approves |

## Rules the server owns

- **Safety:** run an NSFW image check on every generated output and a prompt filter on every request. Rejected jobs return `failed` with a kind message. Teenagers use this.
- **GPU:** `/api/generate` queues behind the GPU lease. Never evict Qwen mid-conversation; report the wait in `message`.
- **Provenance:** when a piece is created, record prompt hash, seed, weights, model, and `ideaBy` to the ledger and return the entry id in `ledger`.
