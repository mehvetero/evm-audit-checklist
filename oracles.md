# Oracle & Price Feed Risks

## Chainlink / Pyth / Band

- [ ] `latestRoundData()` return values are fully validated — check `answeredInRound >= roundId` to catch stale prices
- [ ] Heartbeat timeout is enforced — if the feed has not updated within the expected interval, revert or use a fallback
- [ ] Decimals are handled explicitly — Chainlink feeds return 8 decimals, Pyth returns variable precision, mixing them without normalization causes over/underpayment
- [ ] Feed address is not hardcoded — a proxy feed can be upgraded underneath; verify the aggregator address periodically
- [ ] Sequencer uptime feed is checked on L2s (Arbitrum, Optimism) — prices during sequencer downtime are stale

## TWAP / DEX-Based Oracles

- [ ] TWAP window is long enough to resist manipulation — 30 minutes minimum, 1+ hours preferred for high-value operations
- [ ] Uniswap V3 TWAP uses `observe()` and handles `0` cardinality (uninitialized observations) gracefully
- [ ] Spot price (`getReserves()` / `slot0`) is never used for valuation — trivially manipulable within a single transaction
- [ ] Multi-block manipulation is considered for PoS chains where a validator can control consecutive blocks

## Price Manipulation Scenarios

- [ ] Flash loan → swap (move price) → interact with protocol → swap back — check if any function reads price and acts on it in the same transaction
- [ ] Donation attack — large direct transfer to a pool to skew the reserve ratio before a price read
- [ ] Oracle front-running — MEV bot reads a large trade, updates oracle before the protocol's price read

## Pyth Network Specifics

- [ ] Pyth prices have a confidence interval — high confidence interval means the price is unreliable; do not use `price.price` without checking `price.conf`
- [ ] Pyth uses pull-based updates — the protocol must call `updatePriceFeeds()` before reading; stale data is returned if not updated
- [ ] Pyth `expo` (exponent) is negative — the actual price is `price * 10^expo`; forgetting to handle the exponent produces wildly wrong values
- [ ] Hermes vs on-chain — Pyth Hermes is an off-chain relay; if the Hermes endpoint goes down, prices stop updating
