# Milestone submission - OptimisticClaimOracleV2

Portal → Contribute → **Milestones**. One project may have one milestone in
review at a time, and this is that one.

## Form

| Field | Value |
|---|---|
| 关联项目 | `genlayer-primitives` (the accepted OptimisticClaimOracle project) |
| 里程碑标题 | see below |
| 描述 | the code block below, 987 characters |
| 证明材料 | the URL block below |

## 标题

```
OptimisticClaimOracleV2 - a decided claim is no longer a paid claim
```

## 描述

```
v1 paid out the moment resolve() returned: one adjudication, final. A validator set that misreads a page, or two models sharing one error, moved money permanently - and nobody could appeal an already-paid verdict.

v2 separates deciding from settling. resolve() writes the verdict and opens an appeal window but credits nobody; finalise() pays round one when it closes. appeal() is callable only by the loser, only by matching the bond, and pins up to two counter-evidence URLs. resolve_appeal() reads both document sets and pays 3x bond to the winner, or refunds each stake on INCONCLUSIVE.

Round two is blind: the first verdict, confidence and reasoning never enter the appeal prompt, so an appeal is a fresh reading, not a defence. INCONCLUSIVE stays first-class - a forced verdict is farmable.

30 new tests (86 total): gated transitions, both windows, the 3x payout, refunds, consensus failure, value conservation.

New: optimistic_claim_oracle_v2.py, its design doc, CHANGELOG.md.
```

## 证明材料

```
https://github.com/IanCao/genlayer-primitives
https://github.com/IanCao/genlayer-primitives/blob/main/contracts/optimistic_claim_oracle_v2.py
https://github.com/IanCao/genlayer-primitives/blob/main/CHANGELOG.md
https://github.com/IanCao/genlayer-primitives/blob/main/docs/optimistic_claim_oracle_v2.md
```

The first line is the one the validator reads (repo root = `github-repo`
evidence). The other three are what a human Steward actually wants: the new
contract, the field-by-field diff, and the rationale.

If the portal answers `This URL has already been submitted` - the repo root was
already used by the accepted project submission - say so and the v2 line moves to
its own repository, the same way the escrow did. `CHANGELOG.md` and this file
already contain everything a Steward needs, so the move costs nothing but a push.

## What a Steward will find in the repo

| Question | Where to look |
|---|---|
| What changed, field by field | `CHANGELOG.md` - constructor, states, writes, views, payout, tests |
| Why it changed | `docs/optimistic_claim_oracle_v2.md` - the three v1 failure modes, then the four design choices |
| Is it tested | `tests/test_oracle_v2_logic.py` - 30 tests; `pytest tests -q` shows 86 passed in under a second |
| Is it real code | `contracts/optimistic_claim_oracle_v2.py` - 632 lines, same house style as v1, no imports from it |
| Can I check a deployment | `ORACLE_V2_ADDRESS=0x... node scripts/verify_deployment.mjs` |

## Optional: deploy v2 so the milestone has an on-chain address too

Not required for the submission - the milestone categories it hits are "new
contract functionality", "security or architecture improvement" and "new tests" -
but a live address is the strongest possible evidence and takes one Studio
session.

| Constructor arg | Value |
|---|---|
| `min_bond` | `10000000000000000` (1e16 wei = 0.01 GEN, the protocol floor) |
| `challenge_window` | `86400` (1 day) |
| `appeal_window` | `3600` (1 hour) |

Then verify:

```bash
npm install genlayer-js
ORACLE_V2_ADDRESS=0x<new address> node scripts/verify_deployment.mjs
```

and paste the explorer link
(`https://explorer-studio.genlayer.com/address/0x<new address>`) as a fifth
evidence URL. The first four are enough on their own; this one makes the claim
"it works" unfalsifiable in the good way.
