```text
  ███████╗██████╗ ██╗██████╗ ██████╗ ██████╗   ██╗    ██╗███████╗██████╗ 
  ██╔════╝██╔══██╗██║██╔══██╗╚════██╗██╔══██╗  ██║    ██║██╔════╝██╔══██╗
  ███████╗██████╔╝██║██║  ██║ █████╔╝██████╔╝  ██║ █╗ ██║█████╗  ██████╔╝
  ╚════██║██╔═══╝ ██║██║  ██║ ╚═══██╗██╔══██╗  ██║███╗██║██╔══╝  ██╔══██╗
  ███████║██║     ██║██████╔╝██████╔╝██║  ██║  ╚███╔███╔╝███████╗██████╔╝
  ╚══════╝╚═╝     ╚═╝╚═════╝ ╚═════╝ ╚═╝  ╚═╝   ╚══╝╚══╝ ╚══════╝╚═════╝ 
```

**Spid3rWeb v1.1.0** — Zero-dependency MVC web framework for [Spid3r](https://github.com/needyamin/spid3r) (`v1.1.1+`). Runs on the bytecode VM, tree-walk interpreter, and LLVM 23 AOT compiler.

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

| Command | Purpose |
|---|---|
| `spid3r magic serve [--port N] [--watch] [--tls]` | Start HTTP/HTTPS dev or prod server |
| `spid3r magic build --release --bundle [-o OUT]` | Compile native LLVM 23 AOT binary (`--bytecode-only` for `.matrixc`) |
| `spid3r magic test [--filter=X] [--differential] [--coverage]` | Run 25 test suites (`--db=all` probes PG/MySQL) |
| `spid3r magic doctor [--fix] [--strict]` | Audit runtime, secrets, permissions, ini & DB health |
| `spid3r magic config:check` / `lint` / `fmt [--check]` | Validate 13 configs, static lint, canonical formatting |
| `spid3r magic docs [--json]` / `bench [--gate]` | Generate API docs (`docs/build/`) / run perf gate |
| `spid3r magic route:list` / `key:generate` | Print route table / write 256-bit `APP_KEY` |
| `spid3r magic migrate[:fresh\|:reset\|:rollback\|:status\|:sync]` | Run, inspect, or roll back migrations (`--pretend`, `--step`) |
| `spid3r magic db:seed` / `db:wipe` / `db:backup` / `db:vacuum` | Seed, wipe, back up, or compact DB |
| `spid3r magic queue:work` / `cache:clear` / `cache:gc` | Drain jobs / flush or GC cache |
| `spid3r magic storage:link` / `storage:prune` | Link `public/storage` / prune old logs & temp uploads |
| `spid3r magic make:<kind> <Name>` | Scaffold `controller`, `model [-m]`, `middleware`, `provider`, `request`, `resource`, `command`, `seeder`, `test`, `migration [--fn]` |

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
