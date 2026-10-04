# RPC

Checked: 2026-10-03, background knowledge, `UNVERIFIED-LIVE`. Provider limits and pricing change; read them from the provider's current docs, never from here.

## Commitment
- `processed`: node has seen it in a block; can be rolled back. Fine for low-latency reads, not for accounting.
- `confirmed`: supermajority voted; rollback is rare. Reasonable for operational reads and near-live dashboards.
- `finalized`: rooted; cannot roll back. Use for accounting, reconciliation and anything stored as a result.
Mixed commitments across calls cause phantom disagreements; state which one each number used.

## getSignaturesForAddress paging
- Returns newest first, `limit` default and max 1000.
- `before`: start searching backward from this signature (exclusive). `until`: stop when reaching this signature (exclusive).
- Page: take the last signature of a page as the next `before`. To resume incrementally, pass the newest already-seen signature as `until`.
- Entries include failed txs (`err` non-null). It lists transactions touching the address (incl. as readonly); history depth depends on the node.
- Ordering is by slot, not guaranteed unique per slot ordering across pages; dedupe by signature.

## Costs
- `getTransaction` is one call per signature; `getBlock` returns a whole block (large, heavy, often rate-limited or priced higher). Set `transactionDetails` and `rewards:false` to trim.
- Fetching the broadest version once and storing the raw payload is cheaper than refetching narrower views. Cost scales with calls and payload size, not with how many fields you read.
- Always pass `maxSupportedTransactionVersion: 0` (see `transactions.md`).

## Rate limits and 429
Public endpoints are heavily rate limited and may drop or reorder under load. On HTTP 429: exponential backoff with jitter, honor `Retry-After` if present, cut concurrency, and do not retry in a tight loop. A 429 is not a missing tx. Treat limits as per-provider, per-method facts to read from docs.

## Send is not land
`sendTransaction` returns a signature once the node accepts it (after preflight if enabled). That says nothing about inclusion. Confirm by polling `getSignatureStatuses` (with `searchTransactionHistory` for old ones) until the commitment you need or until `lastValidBlockHeight` passes. Only after expiry plus a finalized-level lookup that still finds nothing can you call it dropped. Durable-nonce transactions do not expire by blockhash height; their fate is decided by whether the nonce account value advanced.

## Slots vs wall time
Slot time targets about 400 ms but is not constant; skipped slots and congestion change it. Convert slot to time with the block's `blockTime` (unix seconds, can be null for some blocks, and is validator-estimated). Do not multiply slots by 0.4 for durations in reports.

## Streaming (gRPC / Geyser / websockets)
Streaming pushes updates as they occur, so cost is by subscription breadth and data volume (accounts, transactions, blocks), usually billed or limited per connection or bandwidth. Broad filters (whole-program accounts, all transactions) can saturate bandwidth and lag the consumer; narrow by account, owner or program. Streams give `processed`/`confirmed` data first; reconcile against `finalized` for accounting. Expect disconnects; resume from a known slot and dedupe by signature.
