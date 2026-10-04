# Fees and tips

Checked: 2026-10-03, background knowledge, `UNVERIFIED-LIVE`. Fee parameters are governed by feature gates and SIMDs; confirm against current Solana docs and the SIMD index for the cluster version.

## Fee components
- **Base fee**: 5000 lamports per signature (historical default; verify against current cluster).
- **Priority fee** = `ceil(compute_unit_price_micro_lamports * compute_unit_limit / 1_000_000)` lamports. It is computed on the **requested** CU limit, not CUs consumed. Over-requesting the limit overpays.
- Total `meta.fee` = base + priority.
- **Fees are charged even when the tx fails** (once it is included in a block and the payer can pay). A failed landed tx costs money and produces a signature.
- Fee-split/burn rules (portion of base fee burned, priority fee to leader) are SIMD-dependent: verify before using them in economics.

## Compute budget instructions
Program `ComputeBudget111111111111111111111111111111`:
- `SetComputeUnitLimit(u32)`: requested CU cap. Without it the default is 200,000 CU per non-builtin instruction, capped at 1,400,000 per tx.
- `SetComputeUnitPrice(u64 micro-lamports per CU)`.
- `RequestHeapFrame`, `SetLoadedAccountsDataSizeLimit`.
Duplicate compute-budget instructions make the tx invalid. Check logs `Program X consumed N of M compute units` for actual use against the limit.

## Atomicity
Instruction effects revert if any instruction fails, **including a tip transfer inside the same tx**. The fee remains charged (and a validated durable-nonce tx also keeps its nonce advance). So "tip paid" must be read from a successful tx (`meta.err == null`), never from the instruction list alone.

## Jito-style tips
- A tip is an ordinary System Program transfer to one of the relay's published tip accounts (list is published by the relay; fetch it live, do not hard-code from memory).
- Bundles are atomic and ordered when processed through the relay's bundle path: all txs land in order in one slot or none do. Exception: if a bundle's block is uncled (skipped slot, block not accepted by the supermajority), any party holding that block can rebroadcast some or all of its transactions individually through the normal path, which does not honor bundle atomicity or revert protection, so they may land or fail separately. A tip must be inside a bundle tx to be paid on bundle success. Source: Jito docs, "Uncled Blocks" (docs.jito.wtf/lowlatencytxnsend).
- **The tip destination does not prove which relay submitted the tx.** Anyone can send a tip transfer by any path, and a bundle's txs look like normal txs on chain. Submission path needs relay-side logs or response IDs, not chain data.
- A tip in a landed, failed tx is reverted; the tip in a non-landed bundle never moves.

## Reading meta.fee vs balance deltas
- Fee payer's SOL delta = `-(fee) + (all other transfers in/out) - (rent paid) + (rent refunded)`. Compute `delta + fee` to see non-fee movement.
- `meta.fee` already includes the priority fee; do not add it again.
- Tips, rent deposits and WSOL wrapping all show up as lamport deltas on the payer or intermediate accounts; classify each before calling anything profit or cost. See `accounts-and-tokens.md`.
- Failed tx: only the payer delta equals `-fee`; any other nonzero delta is a red flag for your account indexing (see the ALT ordering rule in `transactions.md`).
