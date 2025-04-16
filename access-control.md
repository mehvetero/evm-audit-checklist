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



