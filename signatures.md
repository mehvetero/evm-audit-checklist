# Signatures & Replay Protection

## ECDSA / `ecrecover`

- [ ] `ecrecover` return value checked for `address(0)` — a malformed or zero-length signature returns the zero address, not a revert. Without this check, any invalid signature "authenticates" as `address(0)`, and if that address holds permissions the gate is open
- [ ] Signature malleability handled — ECDSA has two valid `(v, r, s)` tuples for every signature. If the contract uses the raw signature as a unique key (e.g., mapping to prevent double-spend), an attacker can flip `s` to replay. Use OpenZeppelin's `ECDSA.recover` which normalizes to the lower-s form
- [ ] `v` value validated — only 27 or 28 are valid for Ethereum. Some `ecrecover` implementations accept other values silently, returning garbage addresses

## EIP-712 Structured Data

- [ ] Domain separator includes `chainId` — without it, a signature on one chain is valid on every chain (mainnet signature replayed on L2 fork)
- [ ] Domain separator includes contract address (`verifyingContract`) — without it, a signature for one contract is valid on any contract with the same struct types
- [ ] Domain separator is recomputed on `chainId` change (or cached with `block.chainid` check) — hard forks that change the chain ID invalidate cached separators. Relevant post-Merge and for L2 forks
- [ ] `DOMAIN_SEPARATOR` is not stored as an immutable if the contract might be deployed on multiple chains — use a function that reads `block.chainid` at call time

## Nonce Management

- [ ] Every signed message includes a nonce — without it, the same signature is valid forever (classic replay)
- [ ] Nonce is incremented atomically with the action — check-then-increment in the same transaction, not in a separate step
- [ ] Nonce is per-signer, not global — a global nonce means one user's transaction invalidates another's pending signature
- [ ] No nonce gap allowed (sequential) or gap handling is explicit — if using non-sequential nonces (bitmap style like Uniswap Permit2), verify the bitmap logic for off-by-one

## Deadline / Expiry

- [ ] Signed messages include a deadline timestamp — without it, a valid signature is replayable indefinitely, even after the signer's intent has changed
- [ ] Deadline comparison uses `<=` or `<` consistently — off-by-one on `block.timestamp == deadline` can lock or unlock a one-second window
- [ ] Signatures cannot be harvested and held — if an operator collects signed messages for batch execution, the deadline limits the operator's window to front-run or withhold

## Permit (ERC-2612)

- [ ] `permit` follows EIP-2612 exactly — non-standard permit implementations (DAI-style with `allowed` mapping) have different semantics and different replay properties
- [ ] `permit` cannot be front-run to cause a revert — if a contract calls `permit` then `transferFrom` in sequence, a front-runner can submit the permit first, making the second `permit` call revert. Use try-catch around `permit` (OpenZeppelin SafeERC20 pattern)
- [ ] Infinite approval via permit (`value = type(uint256).max`) does not decrement allowance — some tokens treat max as "infinite" and skip the decrement, others don't. Check the specific token's implementation

## Multi-Signature

- [ ] Threshold is > 1 for any contract holding significant value — a 1-of-N multi-sig is equivalent to N single-key wallets
- [ ] Signers list cannot be modified by a single signer — adding or removing signers should require the same threshold as execution
- [ ] Transaction hash includes the multi-sig contract address — prevents cross-contract replay between two multi-sigs with the same signers
- [ ] Executed transaction hashes are recorded to prevent replay — `mapping(bytes32 => bool)` for executed hashes

## Real-World References

- **Wintermute (Jun 2022)**: Optimism bridge exploit — 20M OP tokens lost due to incorrect multi-sig setup (1-of-1 threshold during migration)
- **Wormhole (Feb 2022)**: Forged guardian signatures on Solana — `SignatureSet` account was not properly validated, allowing a single guardian to forge a multi-sig quorum
- **Nomad Bridge (Aug 2022)**: Not a signature bug per se, but the proven root was initialized to zero — effectively any message was "signed"
- **Permit front-running**: Multiple DEX aggregators hit by front-runners submitting the user's permit before the aggregator's transaction, causing reverts. Led to the try-catch permit pattern standardized in OpenZeppelin 4.9+
