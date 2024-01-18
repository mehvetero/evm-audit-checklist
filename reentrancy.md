# Reentrancy

## Classic Reentrancy

- [ ] All external calls happen after state updates (checks-effects-interactions pattern)
- [ ] `ReentrancyGuard` from OpenZeppelin is applied to functions that transfer ETH or call external contracts
- [ ] `transfer()` and `send()` are not relied upon for reentrancy protection — gas stipend assumptions break with EIP-1884
- [ ] Callback hooks in received tokens (ERC-777 `tokensReceived`, ERC-721 `onERC721Received`) are considered as reentry points

## Cross-Function Reentrancy

- [ ] Shared state between public functions is protected — function A modifies balance, function B reads it before A completes
- [ ] View functions used in guards do not read stale state during a reentrant call
- [ ] Modifiers that read storage are safe to call mid-execution (e.g., `nonReentrant` checks `_status` which is updated before external call)

## Cross-Contract Reentrancy

- [ ] Protocol-to-protocol calls (e.g., depositing into Aave inside a Compound liquidation) are traced for reentry paths
- [ ] Tokens with transfer hooks (ERC-777, some rebasing tokens) are flagged and tested separately
- [ ] Flash loan callbacks are treated as reentrant by default — the contract's state during the callback may be inconsistent

## Read-Only Reentrancy

- [ ] View functions that return prices or balances are not exploitable via reentrancy into a dependent contract
- [ ] Curve/Balancer-style LP token pricing that reads pool reserves during a swap callback — known read-only reentrancy vector
- [ ] Any `getPrice()` or `getReserves()` used by external protocols is documented as a potential oracle manipulation surface
