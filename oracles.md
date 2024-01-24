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
