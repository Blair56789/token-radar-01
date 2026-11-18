# Token Radar

A token launch radar: it reads a frozen DefiLlama protocol snapshot, ranks every
pre-token protocol by an implied value, and hands the reader a dashboard with a
farm plan for each one. Built for the make.fun token-radar-01 bounty.

## The model (radar-2)

Every row is scored by one function of the snapshot's own figures:

```
implied = round( tvl_usd x comp(category)
                 x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
                 x (0.6 + min(age_days, 1095) / 1095 x 0.4) )
```

`comp(category)` is the median market-cap-to-TVL ratio of the snapshot's own
live-token protocols in the same category (at least 2 of them), else the global
median across all live tokens — clamped to [0.05, 2.0]. A lending token trades
at a different multiple of TVL than a DEX token, and the model prices a
pre-token protocol against its own kind. No protocol name is special-cased.

The board covers all 40 watchlist protocols and ranks every other pre-token
protocol that could still launch a token of its own (190 rows, 150 of them
below the watchlist floor). Wrapped, staked and bridged products of tokens
that already trade, exchanges and versioned pools are set aside, not ranked —
152 of them, listed in the dashboard with the rule that set each aside.
Live-token protocols are set aside with their market caps shown — and they
price the board, as the comparables behind each category multiple.

## Run it

```
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

Writes `board.json`, `data.js`, `dashboard/`, `screenshots/` and
`dossier.json` to the output directory. Deterministic: two runs write the
same bytes.

## Layout

- `main.ts` — the entry program (Node 22, standard library only)
- `lib/model.ts` — the model, the farm plans, the board builder
- `dashboard/` — the page: `index.html`, `app.css`, `app.js` (no network)
- `screenshots/` — three renders of the dashboard: the board, a detail view,
  a filtered view
- `vendor/makefun-board.ts` — make.fun's dossier library, vendored as shipped
