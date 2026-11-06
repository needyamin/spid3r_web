# Spid3rWeb 🕸️

A full-stack **MVC web framework** for the [Spid3r language](https://github.com/needyamin/spid3r). Zero external dependencies — ORM, template engine, migrations, auth, cache, queues, DI container, events, REST resources and CLI all run natively on Spid3r's interpreter and bytecode VM.

📖 **Architecture & roadmap:** [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) · **Reference docs:** [docs/developer-docs.html](docs/developer-docs.html) · **Worked example:** [examples/task-api/main.matrix](examples/task-api/main.matrix)

## Quick Start

**One-click install (Linux / macOS / Windows — Git Bash):**

```bash
sh install.sh            # installs the Spid3r language if missing, then sets up the project
sh install.sh --serve    # ...and starts the dev server immediately
```

**Manual steps** (if Spid3r is already installed):

```bash
spid3r magic migrate     # create schema (migrations auto-discovered)
spid3r magic db:seed     # optional — no-op until you add seeders
spid3r magic serve       # start server → http://localhost:8000
```

A fresh installation serves a single **welcome page** at `/` (`resources/views/welcome.bd.html`). From there, build your app:

| | |
|---|---|
| Add pages | `routes/web.matrix` — `Route.get("/about", ...)` |
| Add templates | `resources/views/*.bd.html` — SpiderBD engine |
| Create code | `spid3r magic make:model Post` · `spid3r magic make:controller PostController` |
| Database | `spid3r magic make:migration create_posts_table` → `spid3r magic migrate` |

## Magic CLI

```bash
spid3r magic serve                    # start the server (optional --port N)
spid3r magic test                     # run all test suites
spid3r magic route:list               # list registered routes
spid3r magic migrate                  # run pending migrations (auto-synced)
spid3r magic migrate:rollback         # roll back the last batch
spid3r magic migrate:status           # show migration status
spid3r magic db:seed                  # seed the database
spid3r magic queue:work               # drain queued jobs (queue:work --once = a single job)
spid3r magic make:controller <Name>   # scaffold a controller
spid3r magic make:model <Name> [-m]   # scaffold a model (+ optional migration)
spid3r magic make:migration <name>    # scaffold a migration (auto-registered)
```

## What's in the Box

| Area | Highlights |
|---|---|
| **HTTP kernel** | keep-alive server loop, graceful shutdown (Ctrl+C), body-size limits, per-request error isolation, static assets with traversal protection |
| **Routing** | `Route.get/post/put/patch/delete`, `:param` captures, groups (prefix + middleware), named routes, 405 detection, `Route.fallback` |
| **Middleware** | onion pipeline + named aliases: `auth` (401 JSON / `/login` redirect), `guest`, `cors`, `logger`, `throttle:60,60` — register your own with `app.middleware("alias", factory)` |
| **Service providers & DI** | `ServiceProvider` base (`register`/`boot`), `Application.register()` + auto idempotent `boot()`, service container (`bind`/`singleton`/`extend`/`alias`), handlers get `req.app` |
| **Events** | global `Event` bus: exact + `prefix.*` wildcard listeners, `once`, class subscribers, `app.booted` lifecycle event |
| **Models & ORM** | Active Record, QueryBuilder (`where`/`whereIn`/joins/unions of clauses), **model lifecycle events with veto**, **batched eager loading** (`load_has_many` / `load_belongs_to` — one query, never N+1), transactions (`DB.transaction(fn)`) |
| **Migrations** | auto-discovery, registry-synced, `make:migration`, rollback/status |
| **Validation** | rule strings (`required|min:3|max:200`, `nullable`, `email`, …), error maps feed 422 responses |
| **Typed exceptions** | `not_found()`, `validation_failed(errors)`, `unauthorized()`, `forbidden()`, `bad_request()`, `too_many_requests()` — thrown anywhere in a handler, rendered centrally (JSON for `/api/*`, HTML cards otherwise). Raw exceptions stay a detail-free 500 |
| **REST resources** | `resource_item` / `resource_collection` / `resource_paginated` envelopes; transformers always receive plain maps (models auto-unwrapped — password columns never leak by construction) |
| **Auth** | bcrypt hashing, CSPRNG session tokens, per-session CSRF tokens, DB-backed sessions with sliding expiry |
| **WebSocket** | `WsServer` — real RFC 6455 server alongside the HTTP app: per-connection protocols **and** cross-connection `broadcast()`/fan-out (safe concurrent send-while-read) |
| **Caching** | memory / file / database drivers, `remember`/`add`/`increment`, powers the throttle middleware |
| **Queues** | database + sync drivers, `routes/queue.matrix` handlers, `queue:work` |
| **Config / Env** | `config/*.matrix` dot-notation + `.env` loading, env-var overrides |
| **CLI** | `magic serve/test/migrate*/db:seed/queue:work/route:list/make:*` |
| **Testing** | 16 suites, `spid3r magic test` exit-code contract |

## Flavor of the Framework

```spid3r
import "framework/Application";
import "framework/HttpException";
import "framework/Resource";

let app = new Application();

Route.get("/api/posts/:id", fn(req) {
    let post = DB.table("posts").find(req.params["id"]);
    if post == null {
        throw not_found("No post with that id.");   // -> 404 JSON for /api/*
    }
    return json(resource_item(post, fn(p) {
        return map("id", p["id"], "title", p["title"]);
    }));
});

app.listen(8000);
```

Providers, model events, and eager loading in [examples/task-api/main.matrix](examples/task-api/main.matrix) — runnable with zero setup:

```bash
SPID3R_DB_DRIVER=sqlite spid3r run examples/task-api/main.matrix
```

## Core Systems

All runtime configuration lives in `config/*.matrix` and is read through a single dot-notation repository:

```spid3r
import "framework/Config/Config";
Config.get("session.lifetime");            // 86400
Config.get("mail.from.address");
```

| System | Facade | Config | Notes |
|---|---|---|---|
| Config | `Config` | `config/*.matrix` | dot paths, `has`/`set`/`load`, env-var overrides |
| Logging | `Log` | `config/logging.matrix` | single + daily channels, levels, `SPID3R_LOG_LEVEL` override, files in `storage/logs/` |
| Filesystem | `Storage` | `config/filesystems.matrix` | `disk("local"/"public")`, `put/get/append/delete/url`, traversal-guarded |
| Mail | `Mail` | `config/mail.matrix` | `log` driver writes `storage/logs/mail.log-*` (SMTP reserved until the runtime gains outbound TCP) |
| Queue | `Queue` | `config/queue.matrix` | `database` driver (`jobs` table) + `sync`; handlers in `routes/queue.matrix`; drain with `magic queue:work` |
| Session | — | `config/session.matrix` | DB-backed; cookie name, lifetime, flags |
| Auth | `Auth` | `config/auth.matrix` | guards, users table/columns, password-reset knobs |
| Services | `Config` | `config/services.matrix` | SMS / payments / OAuth credentials, all env-based |

## Databases

SQLite works with zero config. For PostgreSQL / MySQL / MariaDB, copy `.env.example` → `.env`, set `SPID3R_DB_DRIVER=postgres` (or `mysql` / `mariadb`) and the `SPID3R_PG_*` / `SPID3R_MYSQL_*` values — no code changes needed.

## Performance

The bytecode VM is measurably faster than the tree-walk interpreter on framework work — routing +62%, query compilation +89%, template rendering +114% (see method and numbers in [benchmarks/RESULTS.md](benchmarks/RESULTS.md)). Serve under it in production:

```bash
spid3r run --engine vm server.matrix
```

Reproduce the benchmarks yourself:

```bash
SPID3R_DB_DRIVER=sqlite spid3r run benchmarks/bench.matrix
SPID3R_DB_DRIVER=sqlite spid3r run --engine vm benchmarks/bench.matrix
```

## Layout

```
app/            Your code — Controllers, Models, Middleware (empty on fresh install)
benchmarks/     Micro-benchmarks + recorded results (RESULTS.md)
config/         App, auth, session, cache, database, filesystems, logging, mail, queue, services
database/       Migrations (auto-discovered) + seeders
deploy/         Docker: Dockerfile + docker-compose.yml
docs/           Architecture & roadmap (ARCHITECTURE.md), reference docs, history/
examples/       task-api — runnable full-stack example
framework/      Framework core (kernel, router, ORM, providers, events, resources, exceptions, views, auth, cache, websocket, CLI)
public/         Static assets
resources/      SpiderBD templates (.bd.html)
routes/         web.matrix + api.matrix + queue.matrix (job handlers)
storage/        Logs, cache, compiled views — runtime artifacts
tests/          16 framework test suites — `spid3r magic test`
tools/          Maintenance scripts (resync.matrix rebuilds the migration registry)

install.sh      One-line project + toolchain setup
magic           CLI entrypoint (`spid3r magic serve|test|migrate|...`)
server.matrix   Production server entrypoint (`spid3r run --engine vm server.matrix`)
spider.toml     Package manifest
```

## Testing

```bash
SPID3R_DB_DRIVER=sqlite spid3r magic test    # 16/16 suites, exit code 0/1
```

Live server suites (PostgreSQL / MySQL) activate automatically when `SPID3R_DB_DRIVER` points at a running server.

## License

GPL-3.0-or-later. Built with ❤️ for the Spid3r ecosystem.
