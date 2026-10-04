---
name: solana-chain
user-invocable: false
description: Solana chain semantics for transaction, account, RPC, fee and error questions. Load for any Solana transaction analysis, account or token-balance accounting, RPC usage, fee or tip math, error interpretation, or arb/MEV classification. Chain knowledge only; no project or operator specifics.
---

# solana-chain

Chain-semantics index. Read only the reference you need.

## When to load
Any Solana transaction, account, RPC, fee, tip or error question.

## How to cite
Always name the **program ID and the instruction index** (and inner index when relevant, e.g. `ix 3, inner 2`). A claim about "the swap" or "the error" without program ID + index is not a finding. Also give the signature and slot.

## References (load on demand)
- `references/transactions.md`: message formats, lookup tables, size limit, blockhash expiry, preflight, lock limits.
- `references/fees-and-tips.md`: base and priority fees, tips, atomicity, reading `meta.fee` vs balance deltas.
- `references/accounts-and-tokens.md`: rent, ATAs, Token vs Token-2022, WSOL, token-balance matching, PDAs.
- `references/rpc.md`: commitment, paging, costs, rate limits, send vs land, slots vs time, streaming.
- `references/errors.md`: InstructionError reading, custom codes, common runtime errors.
- `references/mev-classification.md`: telling cyclic arb from withdrawals, sales, unused pools, flashloan custody.

## Version-sensitive facts policy
- Facts that change with cluster releases carry a "Checked" line (date and source). Those written from background knowledge and not re-fetched are marked `UNVERIFIED-LIVE`; confirm against the current cluster version or docs before relying on them for money decisions.
- Third-party skills and references (for example the Solana Foundation solana-dev skill) are cited by pinned commit SHA recorded at vetting time, and are never auto-installed. No SHA is pinned yet; record one here when a reference is vetted.
- Where evidence is missing, say UNKNOWN. Do not fill gaps with plausible values. An RPC miss (null result, no signature found) is unresolved evidence, not proof of absence.

Written 2026-10-03 from background knowledge. 2026-10-03 spot-check against primary docs (solana.com durable-nonces, PDA, SyncNative pages; docs.jito.wtf uncled blocks) confirmed the nonce-failure, PDA-signing, SyncNative-lag and uncled-bundle statements in the references; all other facts remain `UNVERIFIED-LIVE`.
