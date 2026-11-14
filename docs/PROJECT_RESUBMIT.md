# Project resubmission - Bonded Claims

Origin: contribution `6e03a0a1…` `OptimisticClaimOracle - bonded assertions with
bonded disputes`, rejected from **Intelligent Contracts** on 2026/11/07.

> This cannot be accepted under Intelligent Contracts because the submitted
> repository includes a substantial end-user web application with wallet
> integration and complete claim, challenge, adjudication, appeal, and
> withdrawal flows. Please submit the repository under Project instead.

This file is the exact content used on Portal → My submissions → 重新提交 →
select **Projects**. It is a category correction, not new work: the repository,
the deployed contract and the five copied evidence URLs are unchanged.

| Field | Value | Limit |
|---|---|---|
| 项目名称 | Bonded Claims | - |
| 主标签 | 争议解决 | - |
| 标签 1 | 证据评估 | - |
| 标签 2 | 托管资金申领 | - |
| 一句话简介 | see block 02 | 180 |
| 描述 | see block 03 | 1000 |
| 演示视频 | empty (optional) | - |
| 使用步骤 | 5 steps, block 05 | - |
| 预期验证结果 | block 06 | 500 |
| 合约链接 | empty (optional; the form wants `scan.bradbury.genlayer.com`, this project is on `explorer-studio`) | - |
| 网站 | https://iancao.github.io/genlayer-primitives/ | required |
| GITHUB | https://github.com/IanCao/genlayer-primitives | - |
| 证明材料 | 5 URLs copied by the resubmit form, first one is the required GitHub Repository | required |

Note on the name: `Adjudicate` and `Settle` are already published Projects on
this account, so this resubmission uses a distinct name. The repository and the
site are the ones the Steward already looked at.

## 02 一句话简介

```
An end-to-end dApp for bonded factual claims on GenLayer: post, challenge, adjudicate, appeal and withdraw, with a wallet-connected frontend driving real transactions.
```

## 03 描述

```
Bonded Claims is the full dApp behind OptimisticClaimOracle, resubmitted under Projects as the Steward asked: a wallet-connected app with claim, challenge, adjudication, appeal and withdrawal flows is a product, not a single contract. Same repo, same address.

The problem: an oracle that is wrong for free. An author locks a bond with public evidence URLs; an undisputed claim ages out with no model called. A challenger must match the bond. resolve() fetches the documents; validators re-derive and accept only a matching verdict within 25 confidence points. INCONCLUSIVE refunds both; payouts use a pull ledger.

The frontend is no mock: it reads Studionet without a wallet, adds chain 61999 on connect, tracks writes through proposing / committing / revealing / accepted, and drives Finalise, Appeal, Withdraw and Release as real transactions. A schema probe detects v1/v2/v3 and shows only methods the deployed bytecode has.

Four contracts, docs, 86 offline tests, on-chain verifier.
```

## 05 使用步骤

**Step 1 — Open the console**

```
Open https://iancao.github.io/genlayer-primitives/. It reads claims straight from Studionet, so the list renders with no wallet connected.
```

**Step 2 — Connect a wallet**

```
Click "Connect wallet". The app adds GenLayer Studionet (chain id 61999) to the wallet if it is missing, then shows the shortened address.
```

**Step 3 — Post a claim**

```
Write a falsifiable statement, paste one public URL that backs it, keep the bond at 0.01 GEN, and confirm the transaction. Watch the bottom-right panel move through proposing / committing / revealing to accepted.
```

**Step 4 — Challenge it**

```
Switch the wallet to a second account. The new claim shows a "Challenge · 0.01 GEN" button; sending it matches the author's bond exactly and moves the claim to CHALLENGED.
```

**Step 5 — Resolve, then settle**

```
Click "Resolve (round 1)". Validators re-fetch the evidence independently and return a verdict with a confidence score. Nothing is paid yet: either wait out the appeal window and click "Finalise", or let the losing account click "Appeal" and then "Resolve appeal". The pot lands in the winner's credit, and "Withdraw" sends it out.
```

## 06 预期验证结果

```
Step 3: a card #n appears as OPEN, your address as author, bond 0.01 GEN, a countdown, and your evidence link openable.

Step 4: the card reads CHALLENGED with the second account as challenger; the Challenge button is gone.

Step 5: the card shows a verdict (SUPPORTED / REFUTED / INCONCLUSIVE), a confidence out of 100 and one line of reasoning. Nothing moves until Finalise or an appeal; Withdraw then shows the pot. Every write leaves a hash on explorer-studio.genlayer.com.
```

## 07 证明材料 (copied by the resubmit form, unchanged)

```
https://github.com/IanCao/genlayer-primitives
https://github.com/IanCao/genlayer-primitives/blob/main/scripts/verify_deployment.mjs
https://github.com/IanCao/genlayer-primitives/blob/main/docs/optimistic_claim_oracle.md
https://github.com/IanCao/genlayer-primitives/blob/main/contracts/optimistic_claim_oracle.py
https://explorer-studio.genlayer.com/address/0x3a3A9c681cD547ccb293a6329839cc50D529fAd9
```

The first URL is the required `GitHub Repository` evidence. All five were
auto-detected by the form with a green type badge. If the backend answers
`This URL has already been submitted`, these belong to the original rejected
contribution and the fix is a fresh repository root plus fresh deep links -
see `docs/SUBMIT.md` for the rule and the one-repo-per-submission workaround.
