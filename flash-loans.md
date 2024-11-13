# Flash Loan Risks

## Core Checks

- [ ] Any function that reads a balance or price and makes a decision in the same transaction is a flash loan target
- [ ] Protocol's own token balance in a pool can be inflated via flash loan deposit — if `balanceOf(address(this))` is used to determine share price, it is manipulable
- [ ] Flash minting (Aave V3, dYdX) allows borrowing without collateral — trace what the borrowed tokens can do within the callback
- [ ] Compound-style cToken exchange rate uses cash + borrows — cash can be inflated by direct transfer during a flash loan

## Interaction Patterns

- [ ] Deposit → borrow → repay in one transaction — can the protocol be tricked into crediting a deposit that is immediately withdrawn?
- [ ] Liquidation during a flash loan — can an attacker make a position undercollateralized within a single transaction (via price manipulation), liquidate it, and profit?
- [ ] Governance flash loan — can an attacker acquire enough governance tokens via flash loan to pass a proposal in a single block? Check if voting power is snapshot-based or live

## Defenses to Verify

- [ ] Snapshot-based accounting (block.number - 1) for share price or voting power — prevents same-block manipulation
- [ ] Minimum deposit duration before withdrawal — breaks the atomicity needed for flash loan exploitation
- [ ] Rate limiting on large operations — e.g., maximum deposit/withdrawal per block

## Move / Sui Flash Loan Equivalent

Move does not have traditional flash loans. Instead, the "hot potato" pattern enforces atomicity:

```move
struct Receipt { amount: u64 }  // no drop ability

public fun borrow(pool: &mut Pool, amount: u64): (Coin<SUI>, Receipt) { ... }
public fun repay(pool: &mut Pool, coin: Coin<SUI>, receipt: Receipt) { ... }
```

The `Receipt` has no `drop` — the transaction MUST call `repay` or it fails. This is safer than EVM flash loans because the compiler enforces repayment at the type level.

However, the atomic composability through PTBs (Programmable Transaction Blocks) means Move contracts face similar oracle manipulation risks even without traditional flash loans — any price manipulation + action can happen in one transaction.

