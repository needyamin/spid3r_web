# Spid3rWeb 🕸️

A full-stack **MVC web framework** for the [Spid3r language](https://github.com/needyamin/spid3r). Zero external dependencies — ORM, template engine, migrations, auth, cache, admin panel and CLI all run natively on Spid3r's VM.

📖 **Full documentation:** [developer-docs.html](developer-docs.html)

## Quick Start

```bash
spid3r magic migrate     # create schema (migrations auto-discovered)
spid3r magic db:seed     # demo products + admin account
spid3r magic serve       # start server → http://localhost:8000
```

You get **Spid3r Bazaar**, a demo storefront with cart, checkout and an admin panel.

| | |
|---|---|
| Storefront | `/` · product pages `/product/{slug}` · categories `/category/{slug}` |
| Admin panel | `/admin` (dashboard · product CRUD · order status) |
| Demo customer | `ada@example.com` / `secret123` |
| Demo admin | `admin@spid3r.test` / `secret123` — set `SPID3R_ADMIN_EMAIL` in `.env` before seeding for a real admin |

## Magic CLI

```bash
spid3r magic serve                    # start the server (optional --port N)
spid3r magic test                     # run all 19 test suites
spid3r magic route:list               # list registered routes
spid3r magic migrate                  # run pending migrations (auto-synced)
spid3r magic migrate:rollback         # roll back the last batch
spid3r magic migrate:status           # show migration status
spid3r magic db:seed                  # seed the database
spid3r magic make:controller <Name>   # scaffold a controller
spid3r magic make:model <Name> [-m]   # scaffold a model (+ optional migration)
spid3r magic make:migration <name>    # scaffold a migration (auto-registered)
```

## Databases

SQLite works with zero config. For PostgreSQL / MySQL / MariaDB, copy `.env.example` → `.env`, set `SPID3R_DB_DRIVER=postgres` (or `mysql` / `mariadb`) and the `SPID3R_PG_*` / `SPID3R_MYSQL_*` values — no code changes needed.

## Layout

```
app/            Your code — Controllers, Models, Middleware, Support
config/         App & database configuration
database/       Migrations (auto-discovered) + seeders
framework/      Framework core (router, ORM, views, auth, cache, CLI)
public/         Static assets
resources/      SpiderBD templates (.bd.html)
routes/         web.matrix + api.matrix
tests/          Unit + 4-tier E2E suites
```

## License

GPL-3.0-or-later. Built with ❤️ for the Spid3r ecosystem.
