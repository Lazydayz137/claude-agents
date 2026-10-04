# MEV and arb classification

Checked: 2026-10-03, background knowledge; heuristics, not chain facts. Every classification below is a hypothesis scored on evidence. Default to UNKNOWN.

Separate three questions and report each on its own: **route** (does the executed path form a cycle), **execution outcome** (did it land and succeed), **economics** (net result after costs). A cyclic route can execute successfully and still lose money; that is a cyclic route with negative economics, not a non-cycle.

## Cyclic route: criteria 1 and 2 must hold
1. **Same asset in and out**: the actor starts and ends the tx in the same asset (commonly SOL/WSOL or a stable), measured on the actor's own accounts after the WSOL/rent rules in `accounts-and-tokens.md`.
2. **Swaps across at least two distinct pools or venues** that actually changed state: the pools' reserve/vault token balances moved opposite to the actor's, and the swap instructions (cite program ID + index, inner index) executed successfully.
Missing criterion 1 or 2: cyclic route not confirmed. Economics decides profitability, not route class.

**Economics (reported separately, not a route criterion)**: net result in that asset after fee, tip and rent are accounted for separately. Report gross, fee, tip and net as separate numbers; call it profitable only if net is positive. A positive-before-fees path is not a profit.

## Look-alikes to rule out
- **Liquidity withdrawal**: the actor receives both pool tokens and burns an LP token, or a pool program instruction named remove/withdraw; looks like "got two assets for one". Not a trade.
- **Plain sale / one-way swap**: asset A out, asset B in, never returning to A. Not cyclic even if B is later sold in another tx (that is two txs, evaluate separately).
- **Router-listed but unused writable pools**: aggregator routes often list many pools as writable accounts that the executed path never touched. Presence in the account list is not usage. Require a vault balance delta or a swap log/inner instruction for each pool counted.
- **Flashloan custody**: borrowed funds appear to enter and leave the actor within the tx. The borrowed principal is not the actor's capital; profit is what remains after repayment plus fee. Identify the lender program and its borrow/repay instruction pair before reading balances.
- **Wrapping/unwrapping and rent refunds**: SOL movements that are not trades.
- **Self-transfers between the actor's own accounts**: net zero; exclude.
- **Failed txs**: no effects except the fee. A failed arb is a cost, not a trade.

## Method
1. Resolve the full account list (static + ALT loaded) and the actor's owned accounts.
2. List every instruction and inner instruction with program ID and index.
3. Compute per-asset net deltas for the actor (union of pre/post token balances, lamports minus fee).
4. Compute per-pool vault deltas for pools you intend to claim were used.
5. Classify; state the evidence per criterion; otherwise **UNKNOWN**.

## Do not infer
- Intent or identity from tip destination or program choice (see `fees-and-tips.md`).
- Relay or submission route from chain data.
- That two txs from similar-looking wallets are the same operator; that needs separate evidence.
