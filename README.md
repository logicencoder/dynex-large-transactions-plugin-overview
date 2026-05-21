# Dynex Large Transactions — public overview

WordPress plugin that publishes a **live feed of large Dynex (DNX) transfers** on [logicencoder.com](https://logicencoder.com), backed by a dedicated SOL worker that watches the Dynex node and pushes new rows into the site database.

**Implementation (private):** [dynex-large-transactions-plugin](https://github.com/logicencoder/dynex-large-transactions-plugin)

---

## The problem

Dynex whales and high-notional moves are interesting to traders and community members, but block explorers are generic and do not match LogicEncoder’s site design or editorial workflow. Operators need:

- A **threshold-based filter** (only txs above N DNX) without manually refreshing an explorer.  
- A **stable public page** that matches the dark theme and typography of the rest of the crypto tools.  
- A **separation of concerns**: the WordPress host must not open RPC ports to the internet; ingestion should be a one-way push from infrastructure that already runs the Dynex daemon.

Without this plugin, large-tx monitoring would be either manual copy-paste or a separate iframed tool with no brand consistency.

---

## Who benefits

| Audience | Benefit |
|----------|---------|
| **Site visitor** | Readable list of large transfers, search by wallet, optional auto-refresh |
| **Community / trader** | Quick view of velocity, top wallets, and recent whale activity |
| **Operator** | Dashboard widget + settings for API key, cache, and display limits |
| **Infrastructure maintainer** | Clear REST contract between SOL `dnx_large_txs.py` and WordPress |

---

## How this fits the stack

| Piece | Role |
|-------|------|
| **SOL `dnx_large_txs.py`** | Polls Dynex RPC (`getinfo`, block by height), applies DNX threshold, POSTs batch JSON |
| **dynex-large-transactions-plugin** (private) | Stores rows, serves shortcode + REST search |
| **Public page** | WordPress page containing `[dynex_transactions]` |

This is **not** the 0xDNX DHIP rich-list plugin (`dynex-0xdnx-dhip-richlist-plugin`) and **not** the DNX swap webapp — those are separate products with their own monitors.

---

## Capabilities (detailed)

### SOL → WordPress ingest (REST)

**What:** The worker sends `POST /wp-json/dynex/v1/transactions` with header `X-API-Key` and a JSON body `{ "transactions": [ ... ] }`. Each item includes timestamp, block number, hash, from/to wallets, and amount string (e.g. `"12000.00000000 DNX"`).

**Why:** Hostinger must not expose the Dynex wallet RPC to the public internet. Push ingest keeps a single authenticated write path and lets the worker run next to the node on SOL.

**Who benefits:** Operator — key rotation in wp-admin without redeploying Python; infra — clear 400/401 semantics on bad payloads or keys.

### Deduplicated storage

**What:** Rows land in table `wp_dynex_transactions` with `INSERT IGNORE` on unique `tx_hash`.

**Why:** The monitor may re-scan overlapping blocks or retry after network blips; duplicates would inflate counts and break trust in the feed.

**Who benefits:** Visitors seeing accurate counts; operator not cleaning duplicate rows by hand.

### Public shortcode `[dynex_transactions]`

**What:** Embeddable block that renders transaction cards, aggregate stats (volume, velocity, active addresses), collapsible analytics sections, and wallet search. Attributes can override global settings: `limit`, `title`, `show_stats`, `show_search`, `refresh`.

**Why:** Editors control placement (landing page, sidebar column, full-width tool page) without PHP edits.

**Who benefits:** Marketing — one shortcode on the Dynex tools page; visitors — consistent UX with Roboto typography and theme-safe dark styling.

### Address search

**What:** Visitors paste a DNX address; the UI queries `GET /wp-json/dynex/v1/search` with pagination (up to 100 rows per page) and shows sent/received counts and volumes for that wallet.

**Why:** Whale watching often starts from “what did this address do lately?” — not only the global feed.

**Who benefits:** Researchers and holders auditing their own or competitor wallets.

### Live refresh (optional)

**What:** When `refresh` interval &gt; 0, front-end JavaScript polls via AJAX `dynex_check_new` to prepend new cards without full page reload.

**Why:** Large txs are time-sensitive during volatile periods; static HTML would feel stale within minutes.

**Who benefits:** Active visitors during high network activity; operator can disable polling under load by setting interval to 0.

### Load more pagination

**What:** AJAX `dynex_load_more` fetches the next page of cards with nonce verification.

**Why:** Default limit keeps first paint fast; power users can scroll deeper history without loading thousands of rows initially.

**Who benefits:** Visitors on mobile; reduces DB load on first render.

### Admin dashboard widget

**What:** wp-admin home screen widget summarizing total txs, today’s count, latest tx, volume estimates, velocity, top addresses — same metrics family as the public stats panels.

**Why:** Operator verifies ingest health at a glance after logging into WordPress without visiting the public page.

**Who benefits:** Operator on call; quick detection of “worker stopped posting” or empty table.

### Settings page (Settings → Dynex Transactions)

**What:** Configure API key, display limit, refresh interval, title, show/hide stats and search, cache TTL. Documents shortcode usage and attribute overrides.

**Why:** Tuning for performance (cache time) and incident response (rotate API key) must not require code deploy.

**Who benefits:** Operator; support when Cloudflare or object cache layers interact with dynamic content.

### Object caching layer

**What:** Rendered shortcode HTML cached in group `dynex_transactions` for configurable seconds; ingest clears registered cache keys.

**Why:** Aggregate SQL over the full table is expensive on shared hosting; caching keeps public pages fast while staying fresh after each ingest batch.

**Who benefits:** All visitors; Hostinger plan CPU limits respected.

### Public search API (read-only)

**What:** Unauthenticated GET search endpoint for wallet history (no write).

**Why:** Enables future mobile apps or internal tools to reuse the same data without scraping HTML.

**Who benefits:** Developers building dashboards; optional integrations.

---

## Operator checklist

1. Confirm SOL worker is running and threshold matches product expectation (default culture: ~9998 DNX — confirm in worker config).  
2. Match `dynex_api_key` in WordPress to worker secret.  
3. Place shortcode on public page; purge page cache if using LiteSpeed/CDN.  
4. Watch dashboard widget after known large on-chain event — inserted count should increase.

---

## What this overview does not include

- Plugin PHP source or live API keys  
- Dynex node RPC credentials or firewall rules on SOL  
- Full Python source of `dnx_large_txs.py` (lives on SOL archive, not this overview repo)

---

## Related repositories

See [REPOS.md](REPOS.md).

---

## Comparison to sibling Dynex products

| Product | Focus |
|---------|--------|
| **Dynex large transactions** (this) | Threshold-based large native DNX transfers |
| **0xDNX DHIP rich list** | Token holder rankings / DHIP metrics |
| **DNX swap webapp** | Swap UI and pricing context |

Choose the overview that matches the feature you are explaining to users or investors.

---

## FAQ-style notes (useful content)

**Why not query the explorer from the browser?**  
Explorers rate-limit and cannot apply LogicEncoder-specific thresholds or combined stats (velocity, top wallets) in one themed component.

**Can visitors submit transactions?**  
No. Only the authenticated ingest route accepts writes.

**Is historical data backfilled automatically?**  
Only txs the worker observes after deployment (plus whatever batches it sends). Backfill requires a one-off worker run pushing older blocks — operational, not automatic in the plugin alone.

**Does the plugin connect to MEXC or CEX prices?**  
No — on-chain DNX amounts only.

---

## Metrics exposed in the UI (what visitors actually see)

Understanding the dashboard helps interpret whether the worker and database are healthy:

| Metric family | What it tells you |
|---------------|-------------------|
| **Total / today count** | Ingest volume; sudden zero today → worker or API key issue |
| **Total volume (DNX)** | Sum of parsed amount strings — whale activity intensity |
| **Hourly / daily velocity** | Recent tx rate — useful during network stress |
| **Active addresses (24h / 7d / 30d / all)** | Participation breadth beyond single whales |
| **Top amount / top tx count addresses** | Concentration risk and repeat movers |
| **Latest / oldest transaction** | Feed freshness and historical depth |

Collapsible sections keep the public page scannable on mobile while still offering power-user depth.

---

## Threshold semantics (operator + community)

The SOL worker applies a **minimum DNX amount** before a transaction is forwarded (interactive default 9998 DNX in the reference script). The WordPress plugin does not re-filter amount on ingest — it trusts the worker batch. If the public feed feels “too noisy” or “too quiet”, change threshold on SOL, not display limit in WordPress alone.

---

**Made by [logicencoder](https://github.com/logicencoder)**
