# Transactions

Checked: 2026-10-03, background knowledge, `UNVERIFIED-LIVE`. Source to confirm against: Solana docs (transactions, versioned transactions) and the validator release notes for the current cluster version.

## Message formats
- **Legacy**: header, flat account-key list, recent blockhash, instructions. Every account is listed statically.
- **v0**: first byte has the high bit set (`0x80 | version`), adds `addressTableLookups`. Lets a tx reference accounts stored in on-chain address lookup tables (ALTs).
- Unversioned clients that cannot parse v0 fail on those txs.

## Address lookup tables: effective account order
For a v0 tx the combined list used by every account index is, in order:
1. static account keys (signers first, then writable, then readonly, per the header),
2. **loaded writable** keys from all lookups, in lookup order,
3. **loaded readonly** keys from all lookups, in lookup order.

All instruction `accounts` indexes and all `meta.preBalances`/`postBalances`/`preTokenBalances.accountIndex` refer to this combined list. Resolving `message.accountKeys` alone is wrong for v0. `meta.loadedAddresses.{writable,readonly}` gives the loaded parts. Lookup-table signers are impossible: a signer must be a static key.

## getTransaction and versions
`getTransaction` / `getBlock` must pass `maxSupportedTransactionVersion: 0`. Without it, v0 transactions return an error instead of data. Treat that error as "tx exists but you could not read it", never as "tx absent".

## Size limit
- Serialized tx max is **1232 bytes** (IPv6 minimum MTU 1280 minus IP and fragment headers). Each signature costs 64 bytes plus 32 bytes per static key, so many signers and accounts exhaust it quickly; ALTs shrink key cost to 1 byte per loaded account.
- Oversize shows up as a client-side serialization error ("Transaction too large"), or an RPC decode error mentioning `too large` with the byte count and max, before any fee is charged. It never reaches the chain, so there is no signature to look up.
- Larger or "v1" transaction formats: **verify against current cluster version**; do not assume either present or absent.

## Signatures
First signature is the transaction ID. Fee payer is the first signer. Duplicate submissions of the same signed tx return `AlreadyProcessed`; they do not execute twice.

## Recent blockhash expiry
A blockhash is valid for about **150 blocks** after its slot (roughly one to two minutes, not a fixed time). Use `lastValidBlockHeight` from `getLatestBlockhash` and compare to current block height; do not use wall time. Expired: `BlockhashNotFound`. Durable nonces are the alternative for long-lived signing.

## Preflight vs skipPreflight
- Default `sendTransaction` runs a simulation at the RPC node first (at the preflight commitment) and rejects on simulated failure; nothing is broadcast.
- `skipPreflight: true` broadcasts regardless. A tx that would fail can then land and **pay the fee while failing**. Preflight can also give false rejects when the node lags (stale state or blockhash).
- A preflight pass proves nothing about landing.

## Account lock limits
- Each tx may lock a bounded number of accounts (historically 64; a higher limit has been proposed or activated in later versions: **verify against current cluster version**). Exceeding it gives `TooManyAccountLocks`.
- Writable accounts are exclusively locked per slot scheduling; two txs writing the same account cannot run in parallel. The same account listed as writable and readonly counts as writable.
- Compute and heap limits live in `fees-and-tips.md`.
