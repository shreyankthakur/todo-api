# Stage 7 — AI vs Me

## My prompt (written from memory, no peeking at the assignment doc)

> Build me a simple REST API for a to-do list app using Python and FastAPI.
>
> I need basic CRUD:
> - get all tasks
> - get one task by id
> - create a new task (just needs a title)
> - update a task (title or done status)
> - delete a task
>
> Store the tasks in memory, no database. Each task should have an id, title, and done (true/false).
>
> Make sure creating a task returns a 201 status, and deleting returns 204. If someone asks for a
> task that doesn't exist, return a 404 error. If someone tries to create a task without a title,
> return a 400 error.
>
> Also I want Swagger docs so I can test it in the browser.

The AI's code is in `ai-version/main.py`. I ran it on port 8001, fired the same Stage 4 checkpoint
curls at it that I used on my own API, and diffed the two files (`ai_vs_me_diff.txt`).

## What did the AI do better?

Honestly, not much — the core CRUD logic is close to identical to mine, just less defensive.
One small plus: it defaulted `done` to `false` inside the `Task` pydantic model itself
(`done: bool = False`) instead of setting it manually in the handler like I did. Slightly
cleaner, though it also means a client *could* set `done: true` on creation, which my prompt
never said was disallowed but wasn't intended.

## What did it get wrong or quietly ignore?

1. **The 400 error I explicitly asked for didn't happen.** Posting `{}` returned FastAPI's
   default **422 Unprocessable Entity** with `{"detail": [...]}`, not the 400 I asked for.
   The AI only added a manual 400 check for the case where `title` is present but empty
   (`""`) — it didn't override FastAPI's built-in validation for a *missing* title field.
   I caught this exact bug in my own first draft too, which is exactly why I know to look
   for it here.
2. **Error shape is `{"detail": "..."}`, not `{"error": "..."}`.** I said "return a 400 error"
   but never specified the JSON shape, so it used FastAPI's default `detail` key everywhere,
   including on 404s. My own API uses `{"error": "..."}` consistently — a decision I made but
   never told the AI to make.
3. **No seed data.** `GET /tasks` returns `[]` on startup. My prompt said "store tasks in
   memory" but never said to pre-populate anything, so the AI reasonably started empty.
4. **No root `/` or `/health` endpoint.** I never mentioned these in my prompt, so they're
   simply absent. Not wrong, just missing because I forgot to ask.
5. **No validation on `PUT`.** You can send `PUT /tasks/1` with an empty body `{}` and it
   returns `200` with the task unchanged, instead of a `400` like mine does. I only specified
   validation for `POST`, not `PUT`, in my prompt.

## What did my prompt forget to specify — and what did the AI silently decide for you?

- **Exact error response shape** (`error` vs `detail`) — I never specified a key name, so the
  AI just used FastAPI's default everywhere.
- **Whether missing fields should be 400 or FastAPI's native 422** — I said "400 error" in
  prose but didn't say I needed to override FastAPI's default validation behavior to get it
  consistently. The AI took the path of least resistance (manual check only for the empty-string
  case, default 422 for the missing-field case).
- **Seed/starter data** — I said "in memory" but not "pre-filled with 3 examples," so it starts
  empty.
- **Root/health endpoints** — never mentioned, never built.
- **PUT validation rules** — never specified, so none exist.
- **Extras** (filter, search, stats, reset) — obviously absent since I didn't ask for them at all.

## One-sentence takeaway from the rematch

Everything the AI got "wrong" was actually something my prompt left ambiguous or unsaid —
the 422-vs-400 gap in particular is something I'd only catch because I hit the exact same
bug myself while building Stage 3 by hand.

## Rematch (round 2)

Improved prompt, one sentence: *"Also, override FastAPI's default validation so that ANY bad
request — missing title, empty title, or invalid PUT body — returns 400 with a JSON body
shaped exactly like `{"error": "message"}`, not FastAPI's default 422/`detail` format."*
Regenerating with that one line added fixed the 422→400 gap and the error-shape mismatch, which
were the two real (not just cosmetic) bugs found in round 1.
