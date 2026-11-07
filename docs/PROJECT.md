# Project submission - Adjudicate

Portal → Contribute → **Projects** → 项目提交. Fill the seven blocks below in
order; the character counters are the ones the form enforces.

| Field | Limit | Status |
|---|---|---|
| 项目名称 | - | Adjudicate |
| 一句话简介 | 180 | 148 chars |
| 描述 | 1000 | 969 chars |
| 使用步骤 | - | 5 steps |
| 预期验证结果 | 500 | 402 chars |
| 网站 | required | https://adjudicate-oracle.app.workbuddy.host/?sharecode=QO2IQXTGPX |
| GitHub | required | https://github.com/IanCao/genlayer-primitives |
| 证明材料 | required, GitHub Repository | https://github.com/IanCao/genlayer-primitives |

## 01 基本信息

**项目名称**

```
Adjudicate
```

**主标签 / 标签 1 / 标签 2** - pick from the portal's list; the closest matches to
what this is: `Oracles` (primary), `Infrastructure`, `AI`. If `Oracles` is not
offered, use `DeFi` or `Infrastructure` as primary and keep the second tag as
`AI`.

**标志** - any square mark works; the page uses a black-to-indigo gradient tile.
Generate one from `app/index.html`'s `.mark` element, or upload any 512×512 PNG.

## 02 一句话简介 (148 / 180)

```
A claim oracle where being wrong costs money: post a bond, get challenged, get settled by consensus - and no model runs unless someone disputes you.
```

## 03 描述 (974 / 1000)

```
Adjudicate turns a contested factual claim into a priced question. Post a statement with a bond and the URLs backing it; anyone who disagrees matches the bond; GenLayer validators fetch the documents themselves and decide.

The target is the failure that makes most AI oracles unusable in production: a model that is wrong for free. Here both sides have money at stake, the AI is only the tie-breaker, and an undisputed claim never reaches a model - it ages out and the bond comes back.

v2 adds the remedy v1 lacked. resolve() decides but pays nobody; the loser may appeal by matching the bond with counter evidence; the appeal is adjudicated blind, so the first verdict, confidence and reasoning never enter the second prompt. The winner takes the pot, and INCONCLUSIVE refunds every stake, because an oracle forced to pick a side can be farmed.

For grant milestones, prediction markets, moderation appeals - any registry where an assertion must be falsifiable at a cost.
```

## 04 演示视频

Optional and worth doing later: a 60-second screen recording of post → challenge
→ resolve → appeal is the single highest-value extra here. Skip it for the first
submission rather than shipping a rushed one.

## 05 使用步骤

**Step 1 - 标题: Open the app**

```
Open https://adjudicate-oracle.app.workbuddy.host/?sharecode=QO2IQXTGPX. It reads claims straight from Studionet, so the list renders with no wallet connected.
```

**Step 2 - 标题: Connect a wallet**

```
Click "Connect wallet". The app adds GenLayer Studionet (chain id 61999) to the wallet if it is missing, then shows the shortened address.
```

**Step 3 - 标题: Post a claim**

```
Write a falsifiable statement, paste one public URL that backs it, keep the bond at 0.01 GEN, and confirm the transaction. Watch the bottom-right panel move through proposing / committing / revealing to accepted.
```

**Step 4 - 标题: Challenge it**

```
Switch the wallet to a second account. The new claim shows a "Challenge · 0.01 GEN" button; sending it matches the author's bond exactly and moves the claim to CHALLENGED.
```

**Step 5 - 标题: Resolve, then settle**

```
Click "Resolve (round 1)". Validators re-fetch the evidence independently and return a verdict with a confidence score. Nothing is paid yet: either wait out the appeal window and click "Finalise", or let the losing account click "Appeal" and then "Resolve appeal". The pot lands in the winner's credit, and "Withdraw" sends it out.
```

## 06 预期验证结果 (494 / 500)

```
Step 3: a card #n appears as OPEN, your address as author, bond 0.01 GEN, a countdown, and your evidence link openable.

Step 4: the card reads CHALLENGED with the second account as challenger; the Challenge button is gone.

Step 5: the card shows a verdict (SUPPORTED / REFUTED / INCONCLUSIVE), a confidence out of 100 and one line of reasoning. On a v2 contract nothing moves until Finalise or an appeal; Withdraw then shows the pot. Every write leaves a hash on explorer-studio.genlayer.com.
```

**合约链接**

```
https://explorer-studio.genlayer.com/address/0x3a3A9c681cD547ccb293a6329839cc50D529fAd9
```

Add a second deployment link once v2 is live (same form, second entry).

## 07 项目链接

| Field | Value |
|---|---|
| 网站 | https://adjudicate-oracle.app.workbuddy.host/?sharecode=QO2IQXTGPX |
| GitHub | `https://github.com/IanCao/genlayer-primitives` |
| 证明材料 | `https://github.com/IanCao/genlayer-primitives` (type: GitHub Repository) |

## Notes for the reviewer

- The contract is not a wrapper around an LLM call. The model is only invoked
  inside `resolve()`, which requires a bonded challenge; the uncontested path is
  deterministic and free.
- Consensus is re-derivation, not voting: each validator re-fetches the evidence
  and produces its own decision, and the leader is accepted only on verdict
  equality plus a 25-point confidence tolerance.
- 86 offline tests, no network, under a second:
  `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest tests -q`
- The frontend is 3 files and a vendored bundle; `app/README.md` explains how to
  rebuild it.
