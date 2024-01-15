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
