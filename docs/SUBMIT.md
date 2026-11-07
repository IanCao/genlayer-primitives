# Submission texts

Portal → Contribute → **Intelligent Contracts**. Two contracts, two submissions
(one per weekly slot). Every block below is sized for the actual form.

## How to fill the form

| Field | Value |
|---|---|
| 贡献类型 | 开发者 (Developer) |
| 贡献日期 | 2026/11/07 |
| 标题 | the **标题** line of the submission |
| 备注 / 描述 | the code block under **描述** - 938 and 987 characters, under the 1000 limit |
| 证明材料 | paste the URLs one at a time; the portal tags GitHub links itself |

Submit them one at a time. The first is the oracle, the second the escrow.

## Why every URL is per-contract

The portal rejects a proof URL it has already seen (`This URL has already been
submitted`), and it keeps that record across submissions. So the repo root
`https://github.com/IanCao/genlayer-primitives` can be used **once**, on
whichever submission goes first - it is not in either block below.

Each submission instead carries a release tag pinned to the commit it describes:

| Submission | Tag | What the release page shows |
|---|---|---|
| 1. Oracle | `oracle-v1.0.0` | Design, Studionet address, ctor args, verify command, tag-pinned source links |
| 2. Escrow | `escrow-v1.0.0` | Same, for the escrow |

Tagged URLs never move, so a reviewer opening one years later sees the exact
bytecode that was deployed, not `main` as it drifts. All eight links were checked
to return HTTP 200.

If the portal still refuses the tag URL, it is normalising to the repository root
rather than comparing strings. Only then is a second repository needed: the
contracts share nothing at runtime (`deliverable_escrow.py` imports nothing from
`optimistic_claim_oracle.py`), so the escrow can be lifted into its own repo with
`contracts/`, `docs/`, `tests/` and `scripts/` and pushed as-is, no redeploy.

---

## Submission 1 - Optimistic Claim Oracle

**标题** (62 chars)

```
OptimisticClaimOracle - bonded assertions with bonded disputes
```

**描述** (938 / 1000 chars)

```
Oracles that store whatever a model says are broken: a wrong answer costs nothing, so being wrong is rational. This fixes the incentive, not the model.

post_claim() locks a bond above the protocol floor with up to 3 evidence URLs. If nobody challenges during the window, claim_uncontested() refunds it and no model is called - the common path is deterministic. Only a challenge() on a matched bond triggers resolve().

Consensus: the leader fetches evidence, wraps each document in explicit delimiters, and returns canonical JSON. Every validator independently re-fetches and re-derives a decision, accepting only if verdict matches and confidence is within 25 points. Reasoning may differ; only decision fields are compared.

INCONCLUSIVE refunds both bonds: evidence that omits a claim is not evidence against it, and an oracle forced to pick a side can be farmed. Payout is a pull ledger, so a transfer cannot revert the adjudication.
```

**证明材料**

```
https://github.com/IanCao/genlayer-primitives/releases/tag/oracle-v1.0.0
https://explorer-studio.genlayer.com/address/0x3a3A9c681cD547ccb293a6329839cc50D529fAd9
https://github.com/IanCao/genlayer-primitives/blob/oracle-v1.0.0/contracts/optimistic_claim_oracle.py
https://github.com/IanCao/genlayer-primitives/blob/oracle-v1.0.0/docs/optimistic_claim_oracle.md
```

---

## Submission 2 - Deliverable Escrow

**标题** (56 chars)

```
DeliverableEscrow - escrow that settles on "good enough"
```

**描述** (987 / 1000 chars)

```
Escrows in production settle on a binary signal - a signature, a hash, a price - which is why real work still settles off-chain. The real condition is "does this artifact satisfy what we agreed", so acceptance is a settlement value, not a boolean: it maps to basis points, so PARTIAL pays 50% rather than all-or-nothing.

create_escrow() pins criteria in storage; the provider submits one artifact URL; judge() runs round one. The client either lets settle() release it, or escalates with counter evidence; resolve_dispute() runs round two.

Round one is asymmetric: only the provider has a document, so the guard is independent re-derivation. Round two is adversarial: both parties have documents, the prompt must not defer to round one, and counter evidence that ignores the criteria counts for nothing. Both accept the leader only if verdict matches and payout_bps is within 10 points of the validator's read. _coerce() re-pins payout_bps to its verdict, so drift cannot create money.
```

**证明材料**

```
https://github.com/IanCao/genlayer-primitives/releases/tag/escrow-v1.0.0
https://explorer-studio.genlayer.com/address/0x28Ee25d23d30f8E7C53eF54DBf0dF328813E3105
https://github.com/IanCao/genlayer-primitives/blob/escrow-v1.0.0/contracts/deliverable_escrow.py
https://github.com/IanCao/genlayer-primitives/blob/escrow-v1.0.0/docs/deliverable_escrow.md
```

---

## Both contracts, in one line each

| Contract | Address (Studionet) | Constructor args |
|---|---|---|
| OptimisticClaimOracle | `0x3a3A9c681cD547ccb293a6329839cc50D529fAd9` | `min_bond=1e16`, `challenge_window=86400` |
| DeliverableEscrow | `0x28Ee25d23d30f8E7C53eF54DBf0dF328813E3105` | `escalation_window=3600` |

`scripts/verify_deployment.mjs` re-checks both deployments against this repo:
it pulls the source stored on chain and diffs it against `contracts/`, compiles
the repo file and diffs the schema against the deployed one, executes a view
call, and locates the constructor values inside the contract's own storage
slots. Read-only: no private key, no gas. Four checks, all passing.

---

## If a reviewer asks "is this tested?"

`pytest tests -q` runs 56 tests in about 1 second with no network and no Docker.
They cover bond arithmetic, both state machines, permission edges, all three
verdict paths per contract, validator dissent, confidence/payout drift, and
prompt-injection behaviour (a document that tries to dictate the verdict does not
get to dictate the verdict).

The tests run against `tools/genshim`, a minimal stand-in for GenVM, because the
official `genlayer-test` direct runner needs to download the GenVM runtime from
GitHub releases on first use. Consensus-level behaviour that requires a real
multi-node run is the one thing the suite deliberately does not claim to cover.

For anything longer than the 1000-character box, point them at
`docs/optimistic_claim_oracle.md` and `docs/deliverable_escrow.md`.
