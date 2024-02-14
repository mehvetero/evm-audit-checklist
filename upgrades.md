# Upgrade Patterns

## Transparent Proxy (ERC-1967)

- [ ] Admin cannot call implementation functions — transparent proxy separates admin calls from user calls to prevent function selector clashes
- [ ] `upgradeTo()` is restricted to admin multi-sig with timelock
- [ ] Implementation contract has `_disableInitializers()` in constructor
- [ ] Storage layout is verified between upgrades — adding a new variable before existing ones shifts all storage slots downstream
- [ ] No `selfdestruct` or `delegatecall` in implementation — `selfdestruct` kills the proxy, `delegatecall` can overwrite proxy storage

## UUPS (ERC-1822)

- [ ] `_authorizeUpgrade()` has proper access control — if left empty, anyone can upgrade to a malicious implementation
- [ ] The new implementation also has UUPS upgrade logic — upgrading to a contract without `upgradeTo` bricks the proxy permanently
- [ ] `proxiableUUID()` returns the correct storage slot — mismatch breaks upgrade detection

## Diamond (ERC-2535)

- [ ] `diamondCut()` is restricted to admin — unrestricted cut allows adding arbitrary facets
- [ ] Function selectors do not collide across facets — two facets with the same selector causes unpredictable routing
- [ ] `DiamondLoupe` is implemented — allows external inspection of all facets and selectors
- [ ] Storage uses `DiamondStorage` or `AppStorage` pattern — avoids slot collisions between facets

## General Upgrade Risks

- [ ] Timelock on upgrades (48-72 hours minimum for protocols with >$10M TVL)
- [ ] Upgrade event emitted — monitoring tools can catch unexpected upgrades
- [ ] Old implementation cannot be re-initialized after upgrade
- [ ] Storage gaps (`uint256[50] __gap`) in base contracts for future storage additions
