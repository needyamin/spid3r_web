<div align="center">
<pre>
███████╗██████╗ ██╗██████╗ ██████╗ ██████╗   ██╗    ██╗███████╗██████╗&nbsp;
██╔════╝██╔══██╗██║██╔══██╗╚════██╗██╔══██╗  ██║    ██║██╔════╝██╔══██╗
███████╗██████╔╝██║██║  ██║ █████╔╝██████╔╝  ██║ █╗ ██║█████╗  ██████╔╝
╚════██║██╔═══╝ ██║██║  ██║ ╚═══██╗██╔══██╗  ██║███╗██║██╔══╝  ██╔══██╗
███████║██║     ██║██████╔╝██████╔╝██║  ██║  ╚███╔███╔╝███████╗██████╔╝
╚══════╝╚═╝     ╚═╝╚═════╝ ╚═════╝ ╚═╝  ╚═╝   ╚══╝╚══╝ ╚══════╝╚═════╝&nbsp;
</pre>

**Spid3rWeb v1.1.1** — Zero-dependency MVC web framework for [Spid3r](https://github.com/needyamin/spid3r) (`v1.1.1+`). Runs on the bytecode VM, tree-walk interpreter, and LLVM 23 AOT compiler.

</div>

## Quick Start

```bash
sh install.sh --serve       # install pinned Spid3r, configure .env + APP_KEY, migrate, serve
```

Or manually (with Spid3r `v1.1.1+`):

```bash
cp .env.example .env
spid3r magic key:generate
spid3r magic migrate
spid3r magic serve          # http://127.0.0.1:8000 (health: /health)
```

```spid3r
import "framework/Web";

let app = web();
let Post = model("posts", {"fillable": ["title", "body", "published"], "casts": {"published": "bool"}});

app.get("/api/posts", req => Post.latest().paginate(req.page(), req.per_page()));
app.post("/api/posts", (req, res) => {
    let data = req.validate_or_fail({"title": "required|min:3", "body": "required|min:10"});
    return res.status(201).json(Post.create(data).to_map());
}).middleware(["auth", "throttle:60,60"]);

app.listen(8000);
```

## Features

| Area | Highlights |
|---|---|
| **Barrel API** | `framework/Web`: `web()`, `model()`, `router()`, `route()`, `abort()`, `db()`, `auth()`, `jwt()`, `cache()`, `views()`, `test_client()` |
| **Routing** | O(1) static + 2-level prefix buckets, `:param`/`{param}`, `*wildcard`, `Route.resource`, `mount()`, route-model `bind()` |
| **Middleware** | Onion `(req, next) => ...` pipeline: `auth`, `guest`, `cors`, `logger`, `etag`, `request_id`, `throttle` |
| **ORM & DB** | `model()` casts/hidden/dirty tracking, `QueryBuilder`, `Schema`, `ConnectionPool`, SQLite / PostgreSQL / MySQL / MariaDB |
| **Auth & Security** | Argon2id + Bcrypt rehash-on-login, stateless HS256 JWT, DB sessions, 2-layer CSRF, CSP/HSTS, hardened uploads |
| **Views & Static** | SpiderBD v2 templates with `@if`/`@foreach`/`@csrf`/`@method`, chunked streaming, ETag `304`, `Range`, gzip |
| **Systems & Testing** | Cache, Queue, Mail, Log, Storage, Events, DI Container, RFC 6455 WebSocket, in-memory `TestClient`, 25 suites |

## Architecture

<p align="center">
  <img src="docs/diagrams/architecture-overview.svg" alt="Spid3rWeb Architecture Overview" width="100%" />
</p>

**Request flow:** `Client → Auth/JWT → CSRF/Static → Onion Middleware → O(1) Router → Controller/ORM → Response`

| Directory | Role |
|---|---|
| `app/` | Controllers, Providers (and generated Models, Middleware, Requests, Resources, Commands) |
| `bootstrap/` | Paths & application factory (`paths.matrix`, `app.matrix`) |
| `config/` | 13 configuration namespaces (`app`, `database`, `security`, `upload`, ...) |
| `database/` | Migrations & seeders (`database/migrations/`, `database/seeders/`) |
| `framework/` | Core framework (`Web.matrix`, `Http/`, `Foundation/`, `Database/`, `Auth/`, `Console/`, ...) |
| `public/` & `resources/` | Static assets (`/css`, `/js`, `/uploads`) and SpiderBD v2 templates (`views/*.bd.html`) |
| `routes/` | `web.matrix`, `api.matrix`, `queue.matrix` |
| `storage/` | Runtime state: `app/`, `framework/cache/`, `logs/`, `database/`, `backups/` |
| `tests/` | 25 auto-discovered test suites (`spid3r magic test`) |
| `deploy/` | `Dockerfile`, `docker-compose.yml`, `nginx.conf`, `Caddyfile` |

## Magic CLI Commands

```bash
# Serve, Build & Docs
spid3r magic serve                      # HTTP/HTTPS server (--port/--watch/--tls)
spid3r magic build --release --bundle   # Native LLVM-23 AOT binary + assets
spid3r magic build --bytecode-only      # Compiled .matrixc (`spid3r exec`)
spid3r magic docs                       # API docs (docs/build/api.html, --json)
spid3r magic bench [--gate]             # Micro-benchmarks and perf gate

# Verification, Health & Hygiene
spid3r magic test                       # 25 suites (--filter/--differential)
spid3r magic doctor [--fix] [--strict]  # Runtime, perms, APP_KEY, ini & DB
spid3r magic config:check               # Validate all 13 config/*.matrix files
spid3r magic lint                       # `spid3r config validate` + `spid3r check`
spid3r magic fmt [--check]              # Format or verify with `spid3r fmt`
spid3r magic route:list                 # List registered routes and names

# Database, Cache, Queue & Storage
spid3r magic migrate                    # Run pending migrations (--pretend/--step)
spid3r magic migrate:fresh              # Wipe tables & re-run migrations
spid3r magic migrate:reset              # Roll back all executed batches
spid3r magic migrate:rollback           # Roll back last migration batch
spid3r magic migrate:status             # Migration status & checksum drift
spid3r magic migrate:sync               # Rebuild generated migration registry
spid3r magic migrate:rename-legacy      # Convert NNN_* files to timestamps
spid3r magic db:wipe                    # Drop all database tables
spid3r magic db:seed                    # Run database seeders
spid3r magic db:backup                  # Online SQLite backup -> storage/backups/
spid3r magic db:vacuum                  # Checkpoint SQLite WAL & VACUUM
spid3r magic storage:link               # Link public/storage -> storage/app/public
spid3r magic storage:prune              # Prune expired logs & temp uploads
spid3r magic cache:gc                   # Remove expired cache entries
spid3r magic cache:clear                # Flush application cache store
spid3r magic queue:work                 # Drain jobs (--once/--daemon/--sleep=MS)
spid3r magic key:generate               # Write 256-bit APP_KEY in .env

# Generators
spid3r magic make:controller <Name>     # MVC controller
spid3r magic make:model <Name> [-m]     # Model repository (+ optional migration)
spid3r magic make:middleware <Name>     # Onion HTTP middleware
spid3r magic make:provider <Name>       # ServiceProvider
spid3r magic make:request <Name>        # Validation request class
spid3r magic make:resource <Name>       # REST resource transformer
spid3r magic make:command <Name>        # Magic CLI console command
spid3r magic make:seeder <Name>         # Database seeder class
spid3r magic make:test <name>           # TestClient test suite
spid3r magic make:migration <name>      # Timestamped migration (--fn for up/down)
```

## Documentation

| Document | Description |
|---|---|
| [docs/developer-docs.html](docs/developer-docs.html) | Main interactive developer guide |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Architecture, module map & stdlib decisions |
| [docs/SECURITY.md](docs/SECURITY.md) | Threat model, CSRF, sessions & runtime sandbox |
| [docs/DRIVERS.md](docs/DRIVERS.md) | SQLite, PostgreSQL & MySQL/MariaDB support matrix |
| [docs/DEPLOY.md](docs/DEPLOY.md) | AOT binary, Docker, Nginx & Caddy deployment |
| [docs/UPSTREAM_REQUESTS.md](docs/UPSTREAM_REQUESTS.md) | Upstream Spid3r language/runtime requests |
| [docs/diagrams/README.md](docs/diagrams/README.md) | Visual architecture & lifecycle diagrams (SVG + Mermaid) |
| [benchmarks/RESULTS.md](benchmarks/RESULTS.md) | VM & LLVM-23 AOT benchmarks + perf gate floors |
| [examples/task-api/main.matrix](examples/task-api/main.matrix) | Runnable zero-setup MVC API example |

## License

GPL-3.0-or-later.
