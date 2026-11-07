# GenLayer Primitives

Two standalone Intelligent Contract primitives for GenLayer, built around one
idea: **an AI should never be the source of truth, only the tie-breaker in a
dispute that already has structure, evidence and skin in the game.**

| Primitive | File | Core question | Consensus mechanism |
|---|---|---|---|
| Optimistic Claim Oracle | `contracts/optimistic_claim_oracle.py` | "Is this bonded claim supported by public evidence?" | Bonded challenge + leader/validator re-derivation with verdict equality and confidence tolerance |
| Deliverable Escrow | `contracts/deliverable_escrow.py` | "Does this delivered artifact satisfy written acceptance criteria?" | Two-round adjudication: asymmetric single-document review, then adversarial two-document appeal |

Both are Python, both extend `gl.Contract`, both are self-contained (no imports
from each other, no frontend, no off-chain service).

## Why these are primitives rather than demos

A "thin LLM wrapper" asks a model a question and stores the answer. Neither of
these contracts does that:

1. **Money is bound to the decision.** In the oracle, the loser forfeits a bond.
   In the escrow, the verdict maps to a payout in basis points. A drifting model
   cannot silently change who gets paid - see `_coerce`, which re-pins
   `payout_bps` to the canonical value for the verdict whenever the two disagree
   by more than 25 percentage points.
2. **Validators never trust the leader.** Every validator independently
   re-fetches the same documents and re-derives its own verdict, then compares
   only the decision fields (`verdict`, `confidence`, `payout_bps`) under an
   explicit tolerance. Reasoning text is free to differ, which is what makes
   multi-model consensus stable.
3. **The happy path never calls an LLM.** An uncontested oracle claim is settled
   by time alone. An escrow whose client never appeals is settled by
   `payout_bps` from round 1 without a second model call. Cost follows
   disagreement, not volume.
4. **Untrusted input is a first-class concern.** Both contracts cap document
   size and count, wrap every fetched document in explicit data delimiters, and
   instruct the model that content inside those delimiters is data, never
   instruction - with tampering defined as a reason to lower confidence rather
   than a reason to follow orders.
5. **Ambiguity has an exit.** The oracle can return `INCONCLUSIVE`, which refunds
   both bonds. Forcing a binary answer on thin evidence is how oracles become
   cheap to manipulate.
6. **Settlement is pull-based.** Wins accrue to a `credits` ledger and the winner
   withdraws. A payout never pushes value to an address that could revert the
   adjudication that just ran.

## Repository layout

```
contracts/    the two primitives
tests/        50 offline logic tests (pytest)
tools/        genshim - a minimal GenVM stand-in so logic tests run anywhere
docs/         per-primitive design notes, ready to paste into a submission
```

## Running the tests

```bash
python -m venv .venv
.venv/Scripts/activate            # Windows;  source .venv/bin/activate on macOS/Linux
pip install pytest                # add genlayer-test only for direct-mode runs
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest tests -q -p no:cacheprovider
```

**51 tests, <1s, no network, no Docker.**

`tests/` runs the contracts against `tools/genshim/genlayer.py`, a ~250 line
stand-in for GenVM that implements exactly the surface these two contracts touch:
storage containers, value types, the message context, the deterministic clock, and
a leader/validator runner. It is what makes it possible to test bond
arithmetic, permissions, state transitions and output coercion on any machine.

What the shim is **not**: a consensus simulator. `run_nondet_unsafe` there runs
the leader once and asks the validator to accept or reject that result, so
multi-node rotation, pickling and WASI behaviour are out of scope.

The `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1` flag exists because `genlayer-test`
registers a pytest plugin that initialises a localnet client at collection time
(`RPC URL: http://127.0.0.1:4000/api`) even for tests that never touch a network.
For direct-mode runs drop the flag:

```bash
pip install genlayer-test
pytest tests_direct -v        # downloads the GenVM runtime on first run
```

## Test coverage map

| Area | Tests | What is proven |
|---|---|---|
| Oracle parameters | 2 | protocol floor, minimum challenge window |
| Claim posting | 5 | bond floor, empty statement, URL shape, evidence cap, recorded deadline |
| Challenge | 3 | exact bond match, double-challenge rejection, challenger recorded |
| Oracle settlement | 6 | SUPPORTED / REFUTED / INCONCLUSIVE payouts, unknown verdict rejected, confidence clamped |
| Consensus behaviour | 3 | validator dissent and confidence drift block settlement, fenced JSON tolerated |
| Injection | 2 | hostile document cannot set the verdict, unreachable evidence degrades to empty |
| Escrow funding | 6 | self-escrow, dust, empty criteria, zero deadline, window floor, recorded terms |
| Escrow delivery | 4 | provider-only submit, URL shape, state transition, judge-before-delivery rejection |
| Escrow round 1 | 6 | ACCEPT/PARTIAL/REJECT quotes, dissent, payout binding, small drift tolerated |
| Escrow escalation | 3 | client-only, requires JUDGED, records dispute |
| Escrow round 2 | 6 | override, confirm, split, both documents in the prompt (and in order), wrong state rejected |
| Refund path | 2 | deadline guard, delivered escrow cannot be refunded |

## Deploying

1. Open <https://studio.genlayer.com>, connect a wallet that holds testnet GEN
   (Studionet has a built-in faucet: the 💧 button in the account selector).
2. Paste one contract file, then deploy with its constructor arguments:
   - `OptimisticClaimOracle(min_bond, challenge_window)` -
     e.g. `10000000000000000` wei (0.01 GEN) and `86400` (24h, keep it short on
     testnet).
   - `DeliverableEscrow(escalation_window)` - e.g. `3600`.
3. Copy the deployed address from the explorer to link it in a submission.

## What integrators can build on top

- **Oracle** - grant-milestone truthfulness, "did the hackathon actually ship"
  registries, prediction-market resolution with a dispute bond, moderation
  appeals, insurance triggers with a challenge phase.
- **Escrow** - freelance milestones, grant tranches, bug bounties, content
  licensing, SLA credits where "acceptable" is a paragraph of prose rather than
  a boolean.

## Known limits (stated on purpose)

- `withdraw()` sends GEN through the ghost contract, so it needs a live network;
  the direct-mode tests stop at the `credits` ledger.
- Round 2 of the escrow is final. There is no round 3 - an appeal court needs
  juror selection, which needs randomness, which is a separate primitive.
- Both contracts assume evidence URLs are served without authentication. Paywalled
  or JS-only pages will degrade to `INCONCLUSIVE` / `REJECT` rather than guess.
