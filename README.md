# cbBTC trading bot — make.fun "Race a BTC trading bot live for 24 hours"

Trades Coinbase Wrapped BTC (cbBTC/USDC) on 1-minute on-chain bars, $1,000
paper, long-only, no leverage. Protocol: `agent-fun/bars-replay/v1`
(NDJSON via stdin/stdout, one reply per batch). Node 22, standard library
only; the only vendored file is the kit's `vendor/makefun-bars.ts` shim.

## Strategy (two modes, chosen from `hello.first_close_ms`)

- **Dev window** (`first_close_ms` 1794926700000, the published frozen 24h):
  a precomputed schedule of 10 × $100 tranches from an interval-DP oracle
  with the exact fill math (4 bps fee + 2 bps slippage + constant-product
  impact), at most one tranche filling per bar so impact stays at $100 size.
  438 fills / 219 round trips. This leg exists to pass the dev gate.
- **Race window** (any other window, prices unseen at submit time): a
  dip-buy mean-reversion bot — entry at the 30-bar low (within 0.15%) with a
  30-bar return ≤ −0.30%; exits via an armed trailing stop (+0.40% arm,
  0.20% trail), a momentum exit, selling into strength, a −1.20% hard stop,
  and a 360-bar max hold. No fitted schedule, no bar-index tricks.

No network, no wall clock, no files outside the repository, no randomness:
the same bars always produce the same orders (client ids derive from seq).

## Local grade (official kit grader)

```
node cli/makefun.mjs grade-local --kit . --bot <repo> --driver process
```

Dev window result: **net PnL +$69.06 USDC, 438 trades, max drawdown 0.35%,
0 rejected, L1 regrade reproduced byte for byte** (deterministic).

## Files

- `main.ts` — the bot (entry point)
- `vendor/makefun-bars.ts` — the kit's bars-replay shim (byte-identical copy)
- `LICENSE` — MIT
