<img width="492" height="403" alt="image" src="https://github.com/user-attachments/assets/d15ffa79-f3cb-4f7e-9293-504f5cb434d8" /># On-Chain Growth & Marketing Analytics — Uniswap Demo

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
- 4-week wallet retention is **low and variable — ranging from roughly 2% to over 11%, with most cohorts landing in the mid-single digits** — characteristic of an open, permissionless DEX where many wallets trade once and don't return.
- The variation between cohorts is itself a useful signal: in a live setting the next step would be investigating what drove the higher-retention weeks (campaigns, launches, market conditions).

## Notes & limitations

- **TVL** is approximated via trading volume here. True TVL is best sourced from DefiLlama's API or a protocol-specific balances table.
- The most recent retention cohorts read near zero because four full weeks haven't yet elapsed for them.
- For a production setup, off-chain campaign data would be layered in to build full attribution — tying marketing activity to wallet creation and sustained on-chain behaviour.

## Queries

**01 — Daily Active Traders**
```sql
SELECT
  block_date AS day,
  COUNT(DISTINCT taker) AS active_traders,
  COUNT(*) AS trades
FROM dex.trades
WHERE blockchain = 'ethereum'
  AND project = 'uniswap'
  AND block_date >= NOW() - INTERVAL '90' DAY
GROUP BY 1
ORDER BY 1;
```

**02 — Daily Volume & Traders**
```sql
SELECT
  block_date AS day,
  SUM(amount_usd) AS volume_usd,
  COUNT(DISTINCT taker) AS unique_traders,
  COUNT(*) AS trades
FROM dex.trades
WHERE blockchain = 'ethereum'
  AND project = 'uniswap'
  AND block_date >= NOW() - INTERVAL '90' DAY
GROUP BY 1
ORDER BY 1;
```

**03 — Liquidity Activity (TVL proxy)**
```sql
SELECT
  block_date AS day,
  SUM(amount_usd) AS daily_volume_usd,
  AVG(amount_usd) AS avg_trade_size_usd
FROM dex.trades
WHERE blockchain = 'ethereum'
  AND project = 'uniswap'
  AND block_date >= NOW() - INTERVAL '90' DAY
GROUP BY 1
ORDER BY 1;
```

**04 — 4-Week Wallet Retention**
```sql
WITH first_week AS (
  SELECT taker AS trader, DATE_TRUNC('week', MIN(block_date)) AS cohort_week
  FROM dex.trades
  WHERE blockchain = 'ethereum' AND project = 'uniswap'
  GROUP BY taker
),
activity AS (
  SELECT DISTINCT taker AS trader, DATE_TRUNC('week', block_date) AS active_week
  FROM dex.trades
  WHERE blockchain = 'ethereum' AND project = 'uniswap'
)
SELECT
  f.cohort_week,
  COUNT(DISTINCT f.trader) AS cohort_size,
  COUNT(DISTINCT CASE WHEN a.active_week = f.cohort_week + INTERVAL '28' DAY
    THEN a.trader END) AS retained_wk4,
  ROUND(100.0 * COUNT(DISTINCT CASE WHEN a.active_week = f.cohort_week + INTERVAL '28' DAY
    THEN a.trader END) / NULLIF(COUNT(DISTINCT f.trader), 0), 1) AS retention_pct
FROM first_week f
LEFT JOIN activity a ON f.trader = a.trader
WHERE f.cohort_week >= NOW() - INTERVAL '180' DAY
GROUP BY 1
ORDER BY 1;
```

## Stack

Dune Analytics (Trino/DuckDB SQL) · Uniswap on Ethereum · `dex.trades` spellbook tables
