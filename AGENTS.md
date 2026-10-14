# AGENTS.md — fork-observer

## Architecture & Runtime
- `fork-observer` is a Rust application using `warp` (HTTP server + Server-Sent Events) and `rusqlite` (bundled SQLite) with a static D3.js frontend in `./www`.
- The server reads its configuration from `CONFIG_FILE` (defaults to `config.toml`, which is gitignored).
- Frontend assets (`www/index.html`, `www/fullscreen.html`, `www/js/*`, `www/css/*`) are served directly from disk on every request via `warp::fs`, so edits to `./www` take effect immediately without recompiling Rust.
- Rust source changes in `src/` require restarting the `app` service (`docker compose -f docker-compose.base44.yml restart app`), which runs `cargo build --locked && exec cargo run --locked` using cached `/usr/local/cargo` and `/app/target` volumes.

## Configuration Quirks
- `min_fork_height` on each configured network must be kept close to the current chain tip (e.g. `current_tip - 12`) when starting from an empty database with public Electrum/Esplora backends. Setting `min_fork_height = 0` on Mainnet causes non-batch backends (`esplora`) to walk backward block-by-block from the tip (~970,000+ HTTP requests) before writing headers to the database.
- Keeping initial active headers fetched `<= 20` (`tip - min_fork_height <= 20`) also triggers immediate mining pool identification on startup rather than waiting for the 5-minute background sweep.
- When `pool_identification.enable = true`, place an `electrum` node first in `[[networks.nodes]]`: `Esplora::coinbase` in `src/node.rs` has a response variable shadowing bug when fetching raw coinbase tx hex, whereas `Electrum::coinbase` retrieves coinbase transactions cleanly.

## Verification & Testing
- Health/API check: `curl -sf http://localhost:3000/api/networks.json` and `curl -sf http://localhost:3000/api/1/data.json`
- Unit tests: `docker compose -f docker-compose.base44.yml exec -T app cargo test --locked`
