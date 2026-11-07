# Changelog

Not a version-history formality: this file exists so a reviewer can see exactly
what changed between the primitive that was submitted and the one that is
deployed today, without diffing a 400-line contract by eye.

## v2.0.0 - the appeal round

`contracts/optimistic_claim_oracle_v2.py`

v1 settled on a single adjudication. v2 separates deciding from settling and
gives the losing party one round to contest the decision.

### What changed, mechanically

| | v1 | v2 |
|---|---|---|
| Constructor | `(min_bond, challenge_window)` | `(min_bond, challenge_window, appeal_window)` |
| `resolve()` | adjudicates **and pays** | adjudicates, opens the appeal window, pays nobody |
| Payout trigger | `resolve()` | `finalise()` (no appeal) or `resolve_appeal()` |
| States | OPEN, CHALLENGED, SETTLED | + RESOLVED, APPEALED |
| New writes | - | `appeal(claim_id, counter_evidence)`, `resolve_appeal(claim_id)`, `finalise(claim_id)` |
| New views | - | `is_appealable(claim_id)`, `pot_of(claim_id)` |
| Claim fields | 11 | 15 (`appeal_deadline`, `appellant`, `appeal_bond`, `counter_evidence`) |
| Max payout | 2×bond | 3×bond |
| Tests | 26 oracle | 56 oracle (30 new) |

### What did **not** change

- consensus: leader JSON, validators re-fetch and re-derive, verdict equality
  plus a 25-point confidence tolerance;
- the `_coerce_decision` sanitiser and its fence/clamp/validate rules;
- prompt-injection handling: documents are delimited data, tampering lowers
  confidence;
- the pull-payment ledger and `withdraw()`;
- `post_claim`, `challenge`, `claim_uncontested` - same signatures, same guards,
  same messages.

### Why it is a new contract and not an upgrade

v1 is deployed and referenced by an accepted submission, so it is frozen. v2 is a
separate deployment with a different constructor signature; nothing on chain is
mutated, and the two can coexist while integrators migrate.

### Behavioural notes for integrators

- **`resolve()` no longer credits anyone.** A client that used to call
  `resolve()` and then read `get_credit()` must now call `finalise()` after the
  appeal window, or `resolve_appeal()` if an appeal was filed. `is_appealable()`
  and `pot_of()` exist so a UI can show the countdown and the amount at stake.
- **An uncontested claim is unchanged.** `post_claim` + wait + `claim_uncontested`
  still returns the bond, still calls no model, still costs only gas.
- **The appeal window is short by design** (1 hour in the reference deployment).
  It is a window for a party to react, not a second challenge period.
