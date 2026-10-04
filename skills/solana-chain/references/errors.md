# Errors

Checked: 2026-10-03, background knowledge, `UNVERIFIED-LIVE`. Error enums are versioned; confirm names against the runtime and each program's pinned version.

## Reading InstructionError
`{"InstructionError": [index, {"Custom": n}]}` means: the instruction at **`index`** failed with program-specific code `n`.
- `index` is the top-level instruction index. If the failing instruction was reached via CPI, the program that actually raised it may be an inner program: read the logs (`Program <ID> failed: custom program error: 0x..`) to find which program ID emitted it. The log line, not the index alone, identifies the program.
- `Custom n` has no meaning without the program. Hex in logs equals decimal in the JSON (`0x1771` = 6001).
- Always report: program ID, instruction index (and inner index), code in decimal and hex, and the enum name from the program's source or IDL at the deployed version. If you cannot find it: UNKNOWN, not a guess.

## Looking up custom codes
- Use the program's IDL or published error enum at the deployed version (program upgrades can renumber).
- **Anchor**: framework errors are below 6000 (roughly 100-5000: 1xxx deprecated, 2xxx constraint violations, 3xxx account errors such as 3012 AccountNotInitialized, 4100 declared program id mismatch); program-defined errors start at **6000** (first variant = 6000).
- **SPL Token** (when the failing program is the Token program): e.g. 0 NotRentExempt, 1 InsufficientFunds, 4 OwnerMismatch, 17 AccountFrozen. Same number on another program means something else.

## Common runtime (transaction-level) errors
- `BlockhashNotFound`: blockhash expired or unknown to the node. Re-sign with a fresh one; or the node is behind.
- `AlreadyProcessed`: the same signed tx already landed (or is processing); check its status, do not resend blindly.
- `AccountNotFound`: fee payer (or a required account) does not exist on chain / has no funds history.
- `InsufficientFundsForFee`: payer cannot cover fees.
- `InsufficientFundsForRent { account_index }`: a write would leave that account below rent exemption. Index refers to the combined key list.
- `TooManyAccountLocks`: more accounts than the lock limit (see `transactions.md`).
- `AccountInUse`: lock contention with another in-flight tx at scheduling; transient, retry with a fresh blockhash, not a logic bug.
- `AccountLoadedTwice`, `SanitizeFailure`: malformed tx (duplicate keys, bad indexes, bad ALT use); fix construction.
- `ProgramAccountNotFound` / `InvalidProgramForExecution`: program missing, closed or not executable.
- `ComputationalBudgetExceeded` (inner instruction error text "exceeded CUs meter"): the instruction used up its CU limit. Look at logs `consumed N of M`; raise `SetComputeUnitLimit` or reduce work. Also check routing changed between quote and execution.
- `Transaction too large` / `too large: N bytes (max: 1232)`: client-side or RPC decode failure; never landed. Fix with ALTs, fewer accounts or splitting.
- Slippage-type failures are program-specific Custom codes, not runtime errors: find the program's enum.

## What failure implies
- A failed landed tx still pays the fee, and its instruction effects normally roll back, including tips and transfers. Exception: a validated durable-nonce transaction whose execution fails still advances the nonce and collects fees (a nonce tx that fails validation is dropped with no fee; UNVERIFIED-LIVE, not re-fetched from the durable-nonce docs).
- A simulation failure (preflight) says it would fail now, on the node's state, at its commitment; not that it failed on chain.
- No signature found by an RPC query is unresolved evidence, not proof of non-landing: `getTransaction` returns null when the requested commitment has not been reached, and nodes can lack history. Only a client-side failure before send (for example oversize serialization) or blockhash expiry (`lastValidBlockHeight` passed with a finalized-level lookup showing nothing) supports "never landed"; otherwise report UNKNOWN.
