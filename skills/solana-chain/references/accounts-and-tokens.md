# Accounts and tokens

Checked: 2026-10-03, background knowledge, `UNVERIFIED-LIVE`. Program IDs below are the well-known public ones; confirm against current docs before encoding them anywhere.

## Rent
- Every account must hold the rent-exempt minimum for its data size (a 165-byte SPL token account is about 2,039,280 lamports). Depositing it is not a cost; it is refundable on close.
- Closing an account refunds its lamports to the destination named in the close instruction. The refund appears as a positive lamport delta that is not income.
- Creating an account that leaves a balance below the minimum fails with `InsufficientFundsForRent`.

## Associated token accounts (ATA)
- ATA address = PDA of (owner, token program ID, mint) under the ATA program `ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL`. Token vs Token-2022 give different ATAs for the same owner and mint.
- Create (idempotent or not) deposits rent from the payer; closing refunds it to the named destination, often the owner. Count both as rent flow, not trade P&L.

## SPL Token vs Token-2022
- Token: `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA`. Token-2022: `TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb`. Different program IDs; a token account belongs to exactly one.
- Token-2022 mints may carry extensions (transfer fee, transfer hook, confidential transfer, interest-bearing, permanent delegate, non-transferable, etc.). A transfer-fee mint means amount received is less than amount sent; hooks add extra accounts and CPIs. Never assume `sent == received` for Token-2022.
- Token-2022 token accounts can be larger than 165 bytes and need more rent.

## WSOL (wrapped SOL)
- Mint `So11111111111111111111111111111111111111112` (Token program). A WSOL account holds SOL as lamports inside the token account.
- Flow: create/fund the account (System transfer), `SyncNative` to update the token amount, trade, then `CloseAccount` to unwrap (lamports return as SOL).
- **Naive double-count**: the same WSOL principal can appear both as account lamports and as token amount (a wrap shows a SOL delta on the owner and a WSOL token delta; an unwrap shows the opposite). Choose one accounting representation and never count a WSOL account's lamports and its token principal twice. Include persistent (pre-existing and still open), created and closed WSOL accounts; track the refundable rent reserve separately. Reconcile unsynced deposits (token `amount` lags lamports until `SyncNative`), external transfers into the account, and ownership changes before attributing profit. State the rule used.

## Token balance accounting
`preTokenBalances` / `postTokenBalances` entries have `accountIndex`, `mint`, `owner`, and `uiTokenAmount`.
- Match by **(accountIndex, mint)** across the **union** of pre and post. A token account created in the tx has no pre entry (pre = 0); one closed has no post entry (post = 0). Iterating only post (or only pre) drops these.
- `accountIndex` indexes the combined key list including ALT-loaded keys (see `transactions.md`).
- Aggregate to an actor by `owner`, but note that `owner` can change within a tx (`SetAuthority`/assign); use the owner at the time of the question and say which.
- Use raw `amount` (integer) not `uiAmount` (float) for arithmetic.

## PDAs
Program-derived addresses have no private key; only the program whose ID was used to derive the PDA can sign for them (via `invoke_signed`). Signing authority follows the deriving program ID; this is separate from account data ownership (the `owner` field, below), which can differ. A PDA that holds tokens is custody by program logic. Do not attribute a PDA's activity to a wallet. Derivation needs seeds plus program ID; a bump found by search is canonical when highest valid.

## Account ownership
`owner` on an account is the program that may modify its data. System-owned = wallet-type; program-owned = data accounts. Assigning and reallocating accounts changes ownership and rent.
