# Account — balance, plans, transactions

Read-only money tools. The skill NEVER purchases anything — it surfaces balances,
prices, and purchase links; the user buys on the web.

## popcraft_balance `[SPEC]`

- No args. Returns available credits (including any expiring balance) + current plan.
- Call it: at session start when a brief is clearly multi-generation; before quoting a
  plan that would exceed the visible balance; whenever the user asks "how many credits".

## popcraft_show_plans_and_credits `[SPEC]`

- Returns balance + plan + **live pricing with direct purchase links**, as ready-made
  user-facing copy. Pass `intent:"topup"` (out of credits / buy more) or
  `intent:"upgrade"` (plan change / model locked) to order the response for that path.
- **Relay the returned text verbatim** — do not summarize, reorder, or drop the links
  (connector contract). On widget hosts you may add that the pricing card shows the same options.
- This is the designated **recovery step after an insufficient-credits failure**:
  generate fails on credits → call this with `intent:"topup"` → relay → stop (no
  retry until the user confirms they've topped up).

## popcraft_transactions `[SPEC]`

- Recent credit movements (spend / refund / grant), newest first; `limit` ≤50 (default 10).
- Use for: "where did my credits go", verifying a specific charge, and **confirming an
  automatic refund landed after a failed generation** — point to the refund row instead
  of promising amounts yourself.
- **Verify spend by ledger row, not balance delta** — accounts can have concurrent
  sessions charging in parallel; match your job's description + timestamp `[OURS]`.

## Money conversation rules

- Credits are the user's money: cost before spend (R3), actual credits after delivery,
  regeneration-costs-credits noted when offering iteration.
- Never speculate about pricing, discounts, or refund policy from memory — balance and
  plans tools are the only sources.
- Budget-capped briefs ("keep it under N credits"): track a running total across the
  session, report it with each delivery, stop and check in before any call that would cross the cap.
