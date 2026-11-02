# Spid3rWeb 🕸️

A full-stack web framework for the [Spid3r programming language](https://github.com/needyamin/spid3r). Zero external dependencies — runs natively on Spid3r's VM and AOT compiler.

> 📖 Full developer documentation: **[developer-docs.html](developer-docs.html)**

## Features

- **Active Record ORM** — fluent query builder (`DB.table(...)`), timestamps, `fillable` mass-assignment protection, relationships
- **Multi-engine database** — SQLite (default), PostgreSQL, MySQL, MariaDB via `SPID3R_DB_DRIVER`
- **Migrations & seeders** — declarative `Schema.create(...)`, batch tracking, rollback
- **SpiderBD templates** — `.bd.html` layouts with `@extends`, `@section`, `@include`, `@if`, `@foreach`, `{{ $var }}`
- **Expressive router** — static & parameterized routes (`:id`), groups, middleware, 405 handling
- **Middleware pipeline** — CORS, logging, auth guards, custom error pages
- **Built-in auth** — bcrypt hashing, CSPRNG session tokens (cookie or Bearer)
- **Magic CLI** — migrations, seeding, scaffolding, route listing, test runner

## Quick Start

```bash
# 1. Initialize the database
spid3r magic migrate
spid3r magic db:seed

# 2. Start the dev server (default port 8000)
spid3r magic serve
```

Open **http://localhost:8000** — demo login: `ada@example.com` / `secret123`.

### Using PostgreSQL / MySQL / MariaDB

**Option A — `.env` file (recommended):** copy `.env.example` to `.env`, set `SPID3R_DB_DRIVER=postgres` and the `SPID3R_PG_*` values, then just run the commands. Real OS environment variables always take precedence over `.env`.

**Option B — export variables:**

```bash
export SPID3R_DB_DRIVER=postgres          # or: mysql | mariadb
export SPID3R_PG_HOST=127.0.0.1 SPID3R_PG_PORT=5432
export SPID3R_PG_USER=app SPID3R_PG_PASSWORD=secret SPID3R_PG_DATABASE=myapp
```

Then:

```bash
spid3r magic migrate && spid3r magic serve   # app code needs no changes
```

Notes: PostgreSQL requires md5/trust auth; MySQL users require `mysql_native_password`; MariaDB works out of the box. SQLite needs no configuration.

## Magic CLI Commands

| Command | Description |
|---|---|
| `spid3r magic serve [--port N]` | Start the HTTP server |
| `spid3r magic test` | Run the test suite (exit 0/1) |
| `spid3r magic route:list` | List all registered routes |
| `spid3r magic migrate` | Run outstanding migrations |
| `spid3r magic migrate:rollback` | Roll back the last batch |
| `spid3r magic migrate:status` | Show migration status |
| `spid3r magic db:seed` | Seed the database |
| `spid3r magic make:controller <Name>` | Scaffold a controller |
| `spid3r magic make:model <Name> [-m]` | Scaffold a model (optionally + migration) |
| `spid3r magic make:migration <name>` | Scaffold a migration |

## Structure

```
app/            Controllers, Models, Middleware (your code)
config/         App & database configuration
database/       SQLite file, migrations, seeders
framework/      Framework core (router, ORM, views, auth, CLI)
public/         Static assets (css/, js/)
resources/      SpiderBD templates (.bd.html)
routes/         web.matrix + api.matrix route definitions
tests/          Unit, live-driver, and E2E suites
```

## Testing

```bash
spid3r magic test                          # full suite (SQLite)
SPID3R_DB_DRIVER=postgres … spid3r magic test   # suite against a live engine
```

## License

GPL-3.0-or-later. Built with ❤️ for the Spid3r ecosystem.
