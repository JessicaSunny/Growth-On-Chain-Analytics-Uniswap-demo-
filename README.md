# On-Chain Growth & Marketing Analytics — Uniswap Demo

A self-directed project applying marketing-attribution and growth-analytics methods to on-chain data. It tracks how user activity, trading behaviour, and retention evolve on a decentralised exchange — the on-chain equivalent of the campaign-attribution and cohort work I've done in commercial analytics.

**Live dashboard:** https://dune.com/jck_a9563/growth-and-on-chain-analytics-uniswap-demo

Built on Dune Analytics using SQL over Uniswap (Ethereum) data. Uniswap is used as a public demonstration dataset; the same approach transfers directly to any chain or protocol indexed on Dune.

## What it covers

| Panel | What it measures | Growth-analytics parallel |
|-------|------------------|---------------------------|
| Daily Active Traders | Unique trading wallets and trade count per day | User acquisition & engagement trend |
| Daily Volume & Traders | USD trading volume and unique traders over time | Channel/participation volume |
| Liquidity Activity (TVL proxy) | Daily volume as a liquidity-health proxy | Platform depth / supply-side health |
| 4-Week Wallet Retention | Share of new wallets still active 4 weeks after first trade | Acquisition quality / retention cohort |

## Key findings

- Daily active traders rose through April before gradually easing back; trading volume stayed volatile with periodic spikes up to ~$2B.
- 4-week wallet retention is **low and variable — typically 2–3%, with some cohorts reaching 7–9%** — characteristic of an open, permissionless DEX where many wallets trade once and don't return.
- The variation between cohorts is itself a useful signal: in a live setting the next step would be investigating what drove the higher-retention weeks (campaigns, launches, market conditions).

## Notes & limitations

- **TVL** is approximated via trading volume here. True TVL is best sourced from DefiLlama's API or a protocol-specific balances table.
- The most recent retention cohorts read near zero because four full weeks haven't yet elapsed for them.
- For a production setup, off-chain campaign data would be layered in to build full attribution — tying marketing activity to wallet creation and sustained on-chain behaviour.

## Queries

SQL for each panel is in [`/queries`](./queries):

- `01_daily_active_traders.sql`
- `02_daily_volume_traders.sql`
- `03_liquidity_tvl_proxy.sql`
- `04_wallet_retention_4wk.sql`

## Stack

Dune Analytics (Trino/DuckDB SQL) · Uniswap on Ethereum · `dex.trades` spellbook tables
