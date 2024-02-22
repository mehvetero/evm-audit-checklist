# Token Integration

## ERC-20 Quirks

- [ ] Fee-on-transfer tokens — `balanceOf(address(this))` after `transferFrom` may be less than the amount parameter; always use the delta pattern
- [ ] Rebasing tokens (stETH, AMPL) — balance changes without a transfer event; protocols that cache balances will desync
- [ ] Tokens with blocklists (USDC, USDT) — a blocked address cannot send or receive; withdrawal functions must not revert the entire batch if one recipient is blocked
- [ ] Return value not guaranteed — USDT does not return `bool` on `transfer`; use OpenZeppelin `SafeERC20` or check return data length
- [ ] Tokens with permit (ERC-2612) — `permit()` can be front-run; do not revert if `permit` fails because it may have already been used
- [ ] Double-entry point tokens (old Synthetix SNX pattern) — two addresses map to the same balance; depositing via one and withdrawing via the other drains the pool

## ERC-721 / ERC-1155

- [ ] `safeTransferFrom` triggers `onERC721Received` callback — reentry vector
- [ ] `tokenURI` can return off-chain data (IPFS, HTTP) — if the protocol acts on this data, it is an oracle
- [ ] Batch operations in ERC-1155 (`safeBatchTransferFrom`) — verify loop bounds and gas limits

## Native ETH

- [ ] `address.call{value: amount}("")` is preferred over `transfer` or `send` — gas stipend assumptions fail with account abstraction
- [ ] Contracts that receive ETH must have `receive()` or `fallback()` — otherwise ETH sent via `selfdestruct` is stuck
- [ ] `msg.value` in loops — `msg.value` does not decrease per iteration; if a function processes multiple items using `msg.value`, the total check must happen once, not per item
