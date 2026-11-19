# ============================================================================
# Spid3rWeb production-style container image.
#
# Installs the prebuilt Spid3r v1.1.1+ Linux release tarball (x86_64 / aarch64,
# shipping both `spid3r` VM/interpreter and `spid3r-self` AOT compiler) directly
# into debian:bookworm-slim — no Rust build stage required.
#
# The server binds 0.0.0.0 when SPID3R_HOST=0.0.0.0 (framework default stays
# loopback for local dev — see framework/Application.matrix listen()).
# ============================================================================
FROM debian:bookworm-slim AS runtime

RUN apt-get update -qq && apt-get install -y -qq curl ca-certificates >/dev/null \
    && rm -rf /var/lib/apt/lists/*

# Install prebuilt Spid3r toolchain (spid3r + spid3r-self) into /usr/local/bin
ENV SPID3R_INSTALL_DIR=/usr/local/bin
RUN curl -fsSL https://raw.githubusercontent.com/needyamin/spid3r/main/install.sh \
      | sh -s -- --no-modify-path

WORKDIR /var/www/app

# Application code (fresh-install skeleton + full framework core)
COPY magic server.matrix spider.toml ./
COPY app/ app/
COPY config/ config/
COPY database/ database/
COPY framework/ framework/
COPY public/ public/
COPY resources/ resources/
COPY routes/ routes/
COPY tests/ tests/
COPY .env.example .env.example

# Runtime config: env-first. .env is the fallback template; real values come
# from compose `environment:` which override it.
RUN cp .env.example .env \
    && mkdir -p storage/logs storage/framework/cache database \
    && chmod -R u+rw storage

ENV SPID3R_HOST=0.0.0.0 \
    SPID3R_PORT=8000

EXPOSE 8000

# Migrate on boot (idempotent), then serve. `exec` hands PID 1 to the server
# so Ctrl+C / docker stop reach it directly.
CMD ["sh", "-c", "spid3r magic migrate && exec spid3r magic serve --port ${SPID3R_PORT}"]
