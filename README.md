# EVM Audit Checklist

A structured checklist for reviewing EVM smart contracts. Built from notes across multiple audit engagements.

## How to use

Work through each section during a review. Check items off as you verify them. Not every item applies to every contract — skip what is irrelevant but document why.

## Sections

1. [Access Control](access-control.md)
2. [Reentrancy](reentrancy.md)
3. [Oracle & Price Feeds](oracles.md)
4. [Flash Loan Risks](flash-loans.md)
5. [Upgrade Patterns](upgrades.md)
6. [Token Integration](token-integration.md)

## Contributing

PRs welcome. Each checklist item should have a one-line rationale and ideally a link to a real-world exploit that motivates it.

## License

CC BY 4.0

## Real-World References

Each checklist item links to a real incident where the pattern was exploited or nearly exploited. Some notable ones:

- **Reentrancy**: Curve pool read-only reentrancy (Jul 2023, ~$70M)
- **Oracle**: Mango Markets price manipulation (Oct 2022, $114M)
- **Flash Loan**: Euler Finance (Mar 2023, $197M)
- **Access Control**: Ronin Bridge (Mar 2022, $625M — compromised validator keys)
- **Upgrades**: Wormhole (Feb 2022, $320M — uninitialized implementation)
- **Token**: Fee-on-transfer handling failures across multiple DEX forks

## Status

Work in progress. Adding more items as new audit patterns emerge.

