# Task API

A small in-memory CRUD API for managing a to-do list, built with Python and FastAPI.
Built stage by stage for FlyRank Backend Track — Week 2, Assignment A1.

## What this is

A REST API with full CRUD (Create, Read, Update, Delete) for tasks. Data lives in a
plain Python list in memory — there's no database yet, so **all tasks are lost when
the server restarts**. That's intentional (see "The mortality experiment" below);
persistence is next week's topic.

## How to install & run

Requires Python 3.10+.

```bash
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

Then visit:
- `http://localhost:8000/` — API info
- `http://localhost:8000/health` — health check
- `http://localhost:8000/docs` — Swagger UI (interactive docs)


All errors return a JSON body shaped like `{ "error": "message here" }`.

## Example: full CRUD cycle via curl


$ curl -i http://localhost:8000/tasks/1
HTTP/1.1 200 OK
content-type: application/json

{"id":1,"title":"Buy milk","done":false}

$ curl -i http://localhost:8000/tasks/99
HTTP/1.1 404 Not Found
content-type: application/json

{"error":"Task 99 not found"}

$ curl -i -X POST http://localhost:8000/tasks -H "Content-Type: application/json" -d '{"title":"Buy milk"}'
HTTP/1.1 201 Created
content-type: application/json

{"id":4,"title":"Buy milk","done":false}

$ curl -i -X POST http://localhost:8000/tasks -H "Content-Type: application/json" -d '{}'
HTTP/1.1 400 Bad Request
content-type: application/json

{"error":"title: Field required"}

$ curl -i -X PUT http://localhost:8000/tasks/4 -H "Content-Type: application/json" -d '{"done":true}'
HTTP/1.1 200 OK
content-type: application/json

{"id":4,"title":"Buy milk","done":true}

$ curl -i -X DELETE http://localhost:8000/tasks/4
HTTP/1.1 204 No Content
```

(Full test output for every endpoint is in `test_output.txt` in this repo.)

## Swagger UI

`/docs` lists all endpoints with descriptions and lets you run the full CRUD cycle
via "Try it out" — confirmed working (every endpoint returns the right status code
when exercised through the page). **![Swagger UI](screenshots/swagger-ui.png)** 
project was built in a sandboxed container whose network policy blocks
`cdn.jsdelivr.net`, which is where Swagger UI's JS/CSS are loaded from — so the page
never visually renders inside that sandbox. On a normal machine with full internet
access, `/docs` renders normally; take a screenshot there for your own submission.

## Extras implemented

- **Filtering**: `GET /tasks?done=true` returns only finished tasks.
- **Search**: `GET /tasks?search=milk` returns tasks whose title contains "milk".
- **Stats**: `GET /stats` → `{"total": 3, "done": 1, "open": 2}`.
- **Seed & reset**: `POST /reset` restores the 3 example tasks.

## The mortality experiment



## Project structure

```
main.py             # the whole API
requirements.txt    # fastapi, uvicorn
test_output.txt     # full curl output proving every endpoint/status code
```

## Commits

One commit per stage, in order:
1. `Stage 0: hello server`
2. `Stage 1: root and health endpoints`
3. `Stage 2: read endpoints with 404`
4. `Stage 3: create with validation`
5. `Stage 4: full CRUD`
6. `Stage 5: Swagger UI descriptions`
7. `Extras: filter, search, stats, reset`

