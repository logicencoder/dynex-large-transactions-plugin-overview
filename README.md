# Dynex Large Transactions — WordPress plugin

Displays **large Dynex (DNX) on-chain transfers** on logicencoder.com — aggregate stats, searchable history, pagination, and optional auto-refresh. Ingest is push-only from a private chain worker; the plugin never talks to the Dynex node.

Private plugin: [logicencoder/dynex-large-transactions-plugin](https://github.com/logicencoder/dynex-large-transactions-plugin).

## Visitor UI

Shortcode **`[dynex_transactions]`**:

- Header stats panels (totals, averages, recent activity)
- Collapsible transaction sections
- **Address search** with pagination
- Load-more and check-new AJAX without full page reload
- Optional auto-refresh interval

## Ingest worker (operator service)

**`dnx_large_txs.py`** on operator infrastructure:

- Polls local Dynex JSON-RPC (`getblock`, transaction lists)
- Filters transfers above a configurable DNX threshold
- Batches JSON POST to WordPress with shared API key

No separate public backend overview repo — the worker is documented in the private plugin README and `ARCHITECTURE.md`.

## REST API

| Route | Auth | Role |
|-------|------|------|
| `POST /wp-json/dynex/v1/transactions` | `X-API-Key` | Batch ingest from worker |
| `GET /wp-json/dynex/v1/search` | Public | Search by wallet address |

Custom table **`wp_dynex_transactions`** with `INSERT IGNORE` on `tx_hash` for idempotent ingest.

## WordPress admin

**Settings → Dynex Transactions** — API key rotation, display limits, cache TTL. Dashboard widget with aggregate metrics (total volume, tx count, largest transfer).

**AJAX:** `dynex_load_more`, `dynex_check_new` (logged + nopriv), widget refresh, API key update.

Object cache group `dynex_transactions` with configurable TTL; LiteSpeed bypass helpers when present.

## Configuration options

Key options: `dynex_api_key`, `dynex_display_limit`, `dynex_auto_refresh_seconds`, cache TTL — see private plugin for full list.

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
