# AGENTS.md — fork-observer

## Architecture & Runtime
- `fork-observer` is a Rust application using `warp` (HTTP server + Server-Sent Events) and `rusqlite` (bundled SQLite) with a static D3.js frontend in `./www`.
- The server reads its configuration from `CONFIG_FILE` (defaults to `config.toml`, which is gitignored).
- Frontend assets (`www/index.html`, `www/fullscreen.html`, `www/js/*`, `www/css/*`) are served directly from disk on every request via `warp::fs`, so edits to `./www` take effect immediately without recompiling Rust.
- Rust source changes in `src/` require restarting the `app` service (`docker compose -f docker-compose.base44.yml restart app`), which runs `cargo build --locked && exec cargo run --locked` using cached `/usr/local/cargo` and `/app/target` volumes.

## Synchronized Multi-Network Setup
- `docker-compose.base44.yml` configures four live Bitcoin networks (`Mainnet`, `Signet`, `Testnet3`, `Testnet4`), each pairing Electrum and Esplora nodes with `min_fork_height` computed from the live chain tip on startup.
- Initial node queries on each network are staggered by 400ms so the first node fetches the initial header batch into the shared header tree, allowing subsequent nodes on the same network to verify their tips immediately without redundant downloads.

## Verification & Testing
- Health/API check: `curl -sf http://localhost:3000/api/networks.json` and `curl -sf http://localhost:3000/api/1/data.json`
- Unit tests: `docker compose -f docker-compose.base44.yml exec -T app cargo test --locked`
