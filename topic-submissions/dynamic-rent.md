#  Topic Submission

## Presenter
- Name: Igor Durovic
- Affiliation: Anza Labs
- Role: Software Engineer

## Topic Details
### Summary

Dynamic state rent and account compression on Solana: manage total state size with a low-overhead rent mechanism
that includes an integral controller for reactive price adjustments. Rather than deleting delinquent accounts, they
are compressed to a fixed-size hash commitment that allows for account recovery if the original state is provided.

This unlocks:

1. **lower up-front account creation cost**: the rent-exempt model requires a large bond amount at account creation
that is fixed by the protocol. As SOL price increases, the bond amount also increases in real terms. Also, old,
inactive accounts take up significant space indefinitely. Rent and compression solves this by adjusting prices to
match demand and evicting inactive state.
2. **in-memory account storage**: the total state size target can be set such that all active state can be stored
in-memory, improving performance and predictability. Whether or not this will hold up long-term depends on how
demand for state changes over time, but it will at least temporarily remove the need for a chili peppers type
solution for distinguishing between hot vs cold state.

### Technical Overview

- Current state/implementation
  - Today, accounts pay an upfront rent-exempt minimum; ongoing rent pressure is limited, leading to growth in active state and snapshot size. There’s no built-in, data-level compression for arbitrary accounts in `AccountsDB`.
- Relevant SIMDs or existing proposals
  - SIMD-0329 (Last Accessed Slot Tracking): Add `last_accessed_slot` field to account metadata and update on write-lock.
  - SIMD-0341 (v0 Account Compression): introduces a mechanism to compress accounts into a compact on-chain entry containing metadata and a `data_hash`, with decompression to rehydrate when needed. Whether an account is eligible
  for compression is determined by the compression condition, which defaults to `false` for this proposal
  - SIMD-0344 (Dynamic Rent): describes a mechanism for dynamic rent and extends the compression condition to 
  include rent delinquency so that inactive accounts can be compressed/evicted from the active set. 
    - The rent price is determined by an integral controller that adjusts based on state size growth relative to a
    target total state size.
    - Rent is charged when a transaction acquires a write-lock on an account, using `last_accessed_slot` to determine
    the amount.
    - Accounts can lock up extra SOL to cover a rent shortfall to reduce likelihood of delinquency.
    - Delinquency checks are triggered by transactions submitted by off-chain entities, meaning validators don't
    need to scan accounts manually like with the legacy rent system.

## Supporting Materials

- SIMD-0329: https://github.com/solana-foundation/solana-improvement-documents/pull/329
- SIMD-0341: https://github.com/solana-foundation/solana-improvement-documents/pull/341
- SIMD-0344: https://github.com/solana-foundation/solana-improvement-documents/pull/344

## Date's available (Calls are typically on the 3rd Friday of each month)
1. October 17, 2025

## Time Required
[Estimated presentation time: 20–30 minutes]

