# Access Control

## Owner / Admin

- [ ] `onlyOwner` modifier exists and is applied to all privileged functions
- [ ] Owner cannot be set to `address(0)` — check `transferOwnership` for zero-address guard
- [ ] Two-step ownership transfer preferred (`Ownable2Step`) — single-step `transferOwnership` is a rug vector if called with a wrong address
- [ ] `renounceOwnership` is either removed or protected with a timelock — accidental renounce locks admin functions permanently
- [ ] Multi-sig or timelock on owner actions for contracts holding >$1M TVL

## Role-Based (AccessControl)

- [ ] Default admin role is not left as deployer EOA — transfer to a multi-sig after deployment
- [ ] `grantRole` and `revokeRole` are restricted to admin only — verify `onlyRole(DEFAULT_ADMIN_ROLE)`
- [ ] No function uses `tx.origin` for authorization — `tx.origin` can be phished via a malicious contract call chain
- [ ] Role hierarchy is documented — who can grant what, and to whom

## Initializers (Proxies)

- [ ] `initialize()` can only be called once — verify `initializer` modifier from OpenZeppelin
- [ ] Implementation contract has `_disableInitializers()` in constructor — prevents direct initialization of the implementation
- [ ] `initialize()` sets all critical state (owner, fees, oracle addresses) — unset state is a backdoor

## Move Capability-Based Access

The EVM patterns above (onlyOwner, roles) do not apply directly to Move. In Move:

- Admin authority is represented by capability objects (AdminCap, UpgradeCap)
- Access control is enforced at the type level — the function signature requires the capability
- No `msg.sender` equivalent — authorization is "prove you have the object"

Key checks for Move:
- [ ] AdminCap is created in `init()` and transferred to a specific address (not shared)
- [ ] UpgradeCap policy is set appropriately (compatible vs additive vs immutable)
- [ ] No capability is stored inside a shared object (indirect access bypass)
- [ ] Multiple capabilities exist for different privilege levels (not one "god cap" for everything)
- [ ] Authorization return values are asserted, not discarded — `vector::contains()` without `assert!` compiles silently in Move (see below)

### The Discarded Return Value (Typus, $3.44M)

Move-specific trap with no direct EVM equivalent. In Solidity, `require(condition)` is a statement that reverts — you cannot accidentally discard it. In Move, `vector::contains()` returns a `bool` with `drop` ability, so calling it without `assert!` is valid and silent:

```move
// VULNERABLE — Typus Finance, live for 11 months before exploit
vector::contains(&whitelist, &tx_context::sender(ctx));
// bool returned and dropped — authorization never enforced

// FIXED — one keyword
assert!(vector::contains(&whitelist, &tx_context::sender(ctx)), E_UNAUTHORIZED);
```

This is harder to spot than a missing check entirely — a reviewer sees the function name, the arguments, and the comment (`// check authority`) and assumes it works. The code reads correct. Only the compiler output is wrong, and the compiler says nothing.

Move has no `#[must_use]` annotation (Rust does). Until it does, treat any `bool`-returning authorization call without `assert!` as a critical finding.

## Return Value Checks (Cross-Chain Lesson)

This applies to both EVM and Move:

- [ ] Every external call's return value is checked — ERC-20 `transfer()` returning `false` instead of reverting is a classic (SafeERC20 handles this)
- [ ] Calls to `ecrecover` check for `address(0)` — a malformed signature returns zero, not a revert
- [ ] Low-level calls (`.call`, `.delegatecall`) check the boolean return — an unchecked `.call` silently fails
- [ ] In Move: any function returning `bool` for an authorization decision must reach an `assert!` or `if`/`abort` — `drop` ability means the compiler never complains


