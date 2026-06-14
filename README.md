# Dynex Large Transactions — WordPress plugin

Displays **large Dynex (DNX) on-chain transfers** on logicencoder.com — stats panels, searchable history, pagination, and optional auto-refresh. Data is pushed from a private chain monitor; the plugin never talks to the Dynex node directly.

Private plugin: [logicencoder/dynex-large-transactions-plugin](https://github.com/logicencoder/dynex-large-transactions-plugin). Ingest worker: **`dnx_large_txs.py`** (private SOL process) POSTs batches with API key.

## Visitor UI

Shortcode **`[dynex_transactions]`** — collapsible sections, address search, load-more via AJAX, aggregate metrics in the header band.

## REST ingest

- **`POST /wp-json/dynex/v1/transactions`** — authenticated batch ingest from monitor
- **`GET /dynex/v1/search`** — public search endpoint

Public AJAX: `dynex_load_more`, `dynex_check_new` for live updates without full page reload.

## WordPress admin

Settings → Dynex Transactions, dashboard widget with aggregate metrics, API key rotation.

## Monitor role (backend)

Separate Python worker polls local Dynex RPC, filters large transfers, POSTs to WordPress. No standalone backend overview repo — monitor is an operator service linked from this product page.

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
