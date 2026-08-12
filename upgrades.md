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

## Cross-Version State Split (Sui / non-proxy upgrades)

Not all upgrade systems use proxies. On Sui, upgrading a package publishes a new version, but the old version stays callable on the same shared objects. Both versions read and write the same state.

This creates a failure mode that proxy-based systems avoid: **two versions of the same function writing the same storage slot with values from different sources**.

Check for:

- [ ] If the upgrade relocates asset storage (struct field → dynamic field, mapping → separate contract, etc.), the old version's write path to shared accounting variables must be disabled
- [ ] After upgrade, the old version cannot overwrite accounting fields (reserves, balances, supply counters) with values from the old storage location
- [ ] State migration is atomic — no window where both old and new storage exist with divergent balances
- [ ] External integrators (aggregators, routers, other protocols) referencing the old package version are identified and considered — they will keep calling the old code

The EVM equivalent would be: imagine a transparent proxy where the old implementation stays deployed at its original address and can still be called directly (not through the proxy) to modify the same storage. Proxy systems prevent this by construction — all calls go through the proxy. Systems without a single entry point (Sui shared objects, some L2 native account abstraction patterns) must enforce it in the upgrade logic.

Real-world example: BlueMove DEX on Sui (July 2026). Upgrade moved tokens from `pool.token_x` to a dynamic object field. V1 swap kept writing `reserve_x = pool.token_x.value()`. V-latest swap wrote `reserve_x = escrow.token_x.value()`. Attacker called V-latest to mint LP (priced against desynced reserve), then V1 to burn LP (withdrawing from pool.token_x). ~714K SUI drained across 363 pools.



