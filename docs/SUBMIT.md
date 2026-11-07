# Submission texts

Copy-paste blocks for the Portal → Contribute → Intelligent Contracts category.
Two standalone contracts, two submissions (one per weekly slot).

Repo: https://github.com/IanCao/genlayer-primitives

Both contracts are deployed on Studionet. `scripts/verify_deployment.mjs`
re-checks the deployment against this repo in one command: it pulls the source
stored on chain and diffs it against `contracts/`, compiles the repo file and
diffs the schema against the deployed one, executes a view call, and locates the
constructor values inside the contract's own storage slots. No private key, no
gas. All four checks pass for both contracts:

| Contract | Address | Constructor args |
|---|---|---|
| OptimisticClaimOracle | `0x3a3A9c681cD547ccb293a6329839cc50D529fAd9` | `1e16`, `86400` |
| DeliverableEscrow | `0x28Ee25d23d30f8E7C53eF54DBf0dF328813E3105` | `3600` |

Each submission leads with its on-chain address, then the repo, the exact source
file, and the design note.

---

## Submission 1 - Optimistic Claim Oracle

**Title:** OptimisticClaimOracle - bonded assertions with bonded disputes

**One line:** A registry where a claim is free to make until it is wrong: the
author locks a bond, a challenger matches it, and only a contested claim ever
reaches AI consensus.

**Description:**

Oracles that store whatever a model says have a structural flaw: a wrong answer
costs nothing, so wrong answers are rational. This primitive fixes the incentive
rather than the model.

Anyone can `post_claim()` with a bond above the protocol floor and up to three
evidence URLs. The claim sits in a challenge window. If nobody disputes it, the
bond is returned by `claim_uncontested()` and **no model is ever invoked** -
the common path is deterministic and cheap. If someone matches the bond with
`challenge()`, `resolve()` runs an adjudication over the same evidence the author
pinned at posting time.

How consensus is used: the leader fetches the evidence, wraps every document in
explicit data delimiters, and returns a canonical JSON decision. Every validator
independently re-fetches and re-derives its own decision, then accepts only if
the `verdict` matches and `confidence` is within 25 points. Reasoning prose is
free to differ - only decision fields are compared, which is what keeps a
multi-model validator set from being permanently undetermined.

Three verdicts matter here, not two. `INCONCLUSIVE` refunds both bonds, because
evidence that merely fails to mention a claim is not evidence against it, and an
oracle forced to pick a side on thin evidence is an oracle that can be farmed
with unfalsifiable claims.

Money is bound to the outcome: SUPPORTED pays the author both bonds, REFUTED pays
the challenger both, INCONCLUSIVE returns each. Settlement accrues to a pull
payment ledger so a payout can never revert the adjudication that produced it.

Reusable for: grant milestone truthfulness, "did the project actually ship"
registries, prediction-market resolution with a dispute bond, moderation appeals,
insurance triggers with a challenge phase.

Links:
0x3a3A9c681cD547ccb293a6329839cc50D529fAd9 (deployed, Studionet)
https://github.com/IanCao/genlayer-primitives
https://github.com/IanCao/genlayer-primitives/blob/main/contracts/optimistic_claim_oracle.py
https://github.com/IanCao/genlayer-primitives/blob/main/docs/optimistic_claim_oracle.md

---

## Submission 2 - Deliverable Escrow

**Title:** DeliverableEscrow - escrow that settles on "good enough"

**One line:** Milestone escrow whose release condition is written prose, resolved
by a two-round adjudication where the second round is adversarial.

**Description:**

Every escrow in production settles on a binary signal - a signature, a hash, a
price - which is why freelance work, grant tranches and bounties still settle
off-chain. The real condition is "does this delivered artifact satisfy what we
agreed", and this contract treats that as a settlement value rather than a
boolean: acceptance maps to basis points, so `PARTIAL` pays 50% instead of
forcing an all-or-nothing flip.

Lifecycle: `create_escrow()` funds the contract with the criteria pinned in
storage; the provider submits exactly one artifact URL; `judge()` runs round one;
the client either lets `settle()` release round one after the window, or files
`escalate()` with counter evidence; `resolve_dispute()` runs round two and moves
the money. If the provider never delivers, `refund_expired()` makes the client
whole.

How consensus is used - and why the two rounds are different code paths:
round one is asymmetric, only the provider has a document, so the only real guard
is independent re-derivation by every validator. Round two is adversarial: both
parties have documents on the record, and the prompt is explicitly told not to
defer to the first decision and to treat a counter document that does not address
the criteria as worth nothing. Both rounds accept the leader only if `verdict`
matches and `payout_bps` is within 10 percentage points of the validator's own
read - a numeric tolerance on the field that moves money, not an exact string
match.

A model that drifts cannot create money: `_coerce()` re-pins `payout_bps` to the
canonical value for its own verdict (ACCEPT 10000 / PARTIAL 5000 / REJECT 0)
whenever the two disagree by more than 25 points.

Reusable for: freelance milestones, grant tranches, bug bounties, content
licensing, SLA credits - the primitive is `(criteria, deliverable) ->
payout_bps`, so new use cases need new criteria, not a new contract.

Links:
0x28Ee25d23d30f8E7C53eF54DBf0dF328813E3105 (deployed, Studionet)
https://github.com/IanCao/genlayer-primitives
https://github.com/IanCao/genlayer-primitives/blob/main/contracts/deliverable_escrow.py
https://github.com/IanCao/genlayer-primitives/blob/main/docs/deliverable_escrow.md

---

## What to say if a reviewer asks "is this tested?"

`pytest tests -q` runs 56 tests in about 1 second with no network and no Docker.
They cover bond arithmetic, both state machines, permission edges, all three
verdict paths per contract, validator dissent, confidence/payout drift, and
prompt-injection behaviour (a document that tries to dictate the verdict does not
get to dictate the verdict).

The tests run against `tools/genshim`, a minimal stand-in for GenVM, because the
official `genlayer-test` direct runner needs to download the GenVM runtime from
GitHub releases on first use. Consensus-level behaviour that requires a real
multi-node run is the one thing the suite deliberately does not claim to cover.
