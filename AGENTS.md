# AGENTS.md — fork-observer

## Architecture & Runtime
- `fork-observer` is a Rust application using `warp` (HTTP server + Server-Sent Events) and `rusqlite` (bundled SQLite) with a static D3.js frontend in `./www`.
- The server reads its configuration from `CONFIG_FILE` (defaults to `config.toml`, which is gitignored).
- Frontend assets (`www/index.html`, `www/fullscreen.html`, `www/js/*`, `www/css/*`) are served directly from disk on every request via `warp::fs`, so edits to `./www` take effect immediately without recompiling Rust.
- Rust source changes in `src/` require restarting the `app` service (`docker compose -f docker-compose.base44.yml restart app`), which runs `cargo build --locked && exec cargo run --locked` using cached `/usr/local/cargo` and `/app/target` volumes.

## Configured Networks & Backends
- `docker-compose.base44.yml` dynamically generates `/app/config.toml` on startup for all four public Bitcoin networks:
  1. `Mainnet` (id=1): 4 Electrum servers + 2 Esplora REST nodes
  2. `Testnet` (id=2): 2 Electrum servers + 2 Esplora REST nodes
  3. `Testnet4` (id=3): 1 Electrum server + 1 Esplora REST node
  4. `Signet` (id=4): 1 Electrum server + 2 Esplora REST nodes
- `min_fork_height` on each configured network is dynamically initialized to `current_tip - 10` on startup so initial header sync and pool identification complete within seconds.

## Verification & Testing
- Health/API check: `curl -sf http://localhost:3000/api/networks.json` and `curl -sf http://localhost:3000/api/<network_id>/data.json`
- Unit tests: `docker compose -f docker-compose.base44.yml exec -T app cargo test --locked --all-features`
- Format check: `docker compose -f docker-compose.base44.yml exec -T app cargo fmt --all -- --check`
