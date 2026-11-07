# Spid3rWeb 🕸️

A modern, lightweight, high-performance **MVC web framework** for the [Spid3r language](https://github.com/needyamin/spid3r). Zero external dependencies — O(1) indexed router, Onion middleware pipeline, zero-boilerplate ORM, SpiderBD v2 template engine, migrations, JWT & session auth, flash sessions, cache, queues, DI container, events, REST resources, `TestClient`, and CLI all run natively on Spid3r's interpreter and bytecode VM.

📖 **Architecture & roadmap:** [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) · **Master Plan:** [plan.txt](plan.txt) · **Reference docs:** [docs/developer-docs.html](docs/developer-docs.html) · **Worked example:** [examples/task-api/main.matrix](examples/task-api/main.matrix)

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

## Flavor of the Framework (Single-Import Express/Fastify DX)

Build complete apps or REST APIs with a single `import "framework/Web";`:

```spid3r
import "framework/Web";

let app = web();

// Define a model repository in 1 line with casts, hidden fields & fillable guard
let Post = model("posts", {
    "fillable": ["title", "body", "published", "meta"],
    "hidden": ["internal_token"],
    "casts": {"published": "bool", "meta": "json"}
});

// Return maps/lists directly -> auto-serialized as 200 JSON
app.get("/api/posts", req => {
    return Post.latest().paginate(req.page(), req.per_page());
});

// Automatic 404 on missing record via find_or_fail()
app.get("/api/posts/:id", req => {
    return Post.find_or_fail(req.int("id")).to_map();
}).name("posts.show");

// Express-style (req, res) handler + 1-line validate_or_fail() (422 on error)
app.post("/api/posts", (req, res) => {
    let data = req.validate_or_fail({
        "title": "required|min:3|max:200",
        "body":  "required|min:10"
    });
    let post = Post.create(data);
    return res.status(201).json(post.to_map());
}).middleware(["auth", "throttle:60,60"]);

app.listen(8000);
```

Run the self-contained full-stack Task API example with zero setup:

```bash
SPID3R_DB_DRIVER=sqlite spid3r run --engine vm examples/task-api/main.matrix
```

## Magic CLI

```bash
spid3r magic serve                    # start the server (optional --port N)
spid3r magic test                     # auto-discover and run all 17 test suites
spid3r magic route:list               # list registered routes and names
spid3r magic migrate                  # run pending migrations (auto-synced)
spid3r magic migrate:rollback         # roll back the last batch
spid3r magic migrate:status           # show migration status
spid3r magic db:seed                  # seed the database
spid3r magic queue:work               # drain queued jobs (queue:work --once = a single job)
spid3r magic cache:clear              # flush the application cache store
spid3r magic key:generate             # generate a random 256-bit APP_KEY in .env
spid3r magic make:controller <Name>   # scaffold a modern MVC controller
spid3r magic make:model <Name> [-m]   # scaffold a zero-boilerplate model (+ optional migration)
spid3r magic make:middleware <Name>   # scaffold an Onion HTTP middleware class
spid3r magic make:provider <Name>     # scaffold a ServiceProvider class
spid3r magic make:test <name>         # scaffold a TestClient integration test suite
spid3r magic make:migration <name>    # scaffold a migration (auto-registered)
```

## What's in the Box

| Area | Highlights |
|---|---|
| **Single-import barrel** | `import "framework/Web";` exports `web()`, `router()`, `model()`, `route()`, `abort()`, `db()`, `schema()`, `cache()`, `auth()`, `jwt()`, `views()`, `test_client()` |
| **HTTP kernel** | keep-alive server loop, graceful shutdown (Ctrl+C), static asset ETag (`304 Not Modified`), method spoofing (`_method` / `X-HTTP-Method-Override`), auto-coercion of map/list/string returns |
| **O(1) & Prefix-Indexed Router** | **133,333 ops/sec** on VM; O(1) hash lookup for static routes, 2-level segment + literal-prefix indexing for `:param` / `{param}` routes, `*wildcard` catch-alls, `app.mount(prefix, sub_router)`, reverse routing via `route(name, params)` |
| **True Onion Middleware** | `(req, next) => { let res = next(req); ... return res; }` closures and class middleware; built-ins: `auth`, `guest`, `cors`, `logger`, `etag`, `request_id`, `throttle:60,60` (with `X-RateLimit-*` headers) |
| **Zero-Boilerplate ORM** | `model("table", opts)` repository + `Model` base class, `casts` (`int`, `float`, `bool`, `string`, `json`), `hidden` column stripping, dirty tracking (`is_dirty()`), `find_or_fail()`, `first_or_fail()`, `update_or_create()`, batched eager loading (`load_has_many` / `load_belongs_to`), mutex-serialized transactions |
| **Validation & Coercion** | `req.validate_or_fail(rules)` returns coerced values or aborts with 422 JSON; rules include `required`, `nullable`, `int`, `float`, `bool`, `email`, `url`, `uuid`, `regex`, `between`, `in`, `unique:table,col`, `exists:table,col` |
| **Auth, JWT & Sessions** | DB-backed sessions + stateless HS256 `Jwt.sign` / `Jwt.verify` (`Bearer` tokens verified with zero DB hits), `req.session()` key-value store + one-time `flash()` messages, HMAC-signed cookies (`res.signed_cookie` / `req.signed_cookie`), multipart file uploads (`req.file()`, `req.store_file()`) |
| **SpiderBD v2 Views** | Pre-compiled template line cache with `file_mtime` invalidation, `@if(a == b and c >= 10)` / `@elif(...)` / `@else`, inline `@if(...)...@endif`, `@foreach` with `loop` metadata (`index`, `iteration`, `first`, `last`, `count`), `@csrf`, `@method('PUT')`, `views().share(k, v)` |
| **Testing (`TestClient`)** | Fluent integration client (`app.test_client().acting_as(user).post_json(...).assert_created().assert_json_path(...)`) |
| **Thread-Safe Core** | Mutex-guarded `MemoryCacheStore` (with bounded capacity eviction), mutex-guarded `Log` file appends, and mutex-serialized `DatabaseManager.transaction()` |

## Performance

See [benchmarks/RESULTS.md](benchmarks/RESULTS.md) for methodology and full breakdown:

- **Route matching (200 routes, `:param` capture):** **133,333 ops/sec** on `--engine vm` (153× faster than the legacy linear router).
- **JSON round-trip:** **333,333 ops/sec** on `--engine vm`.
- **Query compilation:** **20,547 ops/sec** on `--engine vm`.
- **SpiderBD v2 template rendering:** **9,569 ops/sec** on `--engine vm`.

Serve under the bytecode VM in production:

```bash
spid3r run --engine vm server.matrix
```

## Testing

```bash
SPID3R_DB_DRIVER=sqlite spid3r magic test               # 17/17 suites (tree-walk)
SPID3R_DB_DRIVER=sqlite spid3r magic test --engine=vm   # 17/17 suites (bytecode VM)
```

## License

GPL-3.0-or-later. Built with ❤️ for the Spid3r ecosystem.
