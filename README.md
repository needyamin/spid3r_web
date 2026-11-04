# Spid3rWeb 🕸️

A full-stack **MVC web framework** for the [Spid3r language](https://github.com/needyamin/spid3r). Zero external dependencies — ORM, template engine, migrations, auth, cache, config, logging, mail, queue and CLI all run natively on Spid3r's VM.

📖 **Full documentation:** [developer-docs.html](developer-docs.html)

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

The layout is mobile-first with plain semantic CSS — one self-contained stylesheet (`public/css/welcome.css` on a fresh install), no CSS framework, no CDN, no build step.

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

## Core Systems

All runtime configuration lives in `config/*.matrix` and is read through a
single dot-notation repository:

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

```spid3r
import "framework/Queue/Queue";
Queue.push("mail.send", map("to", "ada@example.com", "subject", "Welcome", "body", "Hi!"));
```

## Databases

SQLite works with zero config. For PostgreSQL / MySQL / MariaDB, copy `.env.example` → `.env`, set `SPID3R_DB_DRIVER=postgres` (or `mysql` / `mariadb`) and the `SPID3R_PG_*` / `SPID3R_MYSQL_*` values — no code changes needed.

## Layout

```
app/            Your code — Controllers, Models, Middleware (empty on fresh install)
config/         App, auth, session, cache, database, filesystems, logging, mail, queue, services
database/       Migrations (auto-discovered) + seeders (no-op on fresh install)
framework/      Framework core (router, ORM, views, auth, cache, config, log, mail, queue, CLI)
public/         Static assets
resources/      SpiderBD templates (.bd.html) — welcome.bd.html on fresh install
routes/         web.matrix + api.matrix + queue.matrix (job handlers)
tests/          Framework test suites (9 suites, 9/9 green on SQLite · MySQL 8.0 · MariaDB 11 · PostgreSQL 16)
```

## License

GPL-3.0-or-later. Built with ❤️ for the Spid3r ecosystem.
