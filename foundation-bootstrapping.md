# Foundation Bootstrapping Guide

This document describes the **one‑time setup** required to initialize the BBS Coin Foundation on a given chain.

## 1. Create Safe Multisig
- Choose signer set (e.g. 5 signers)
- Choose threshold (e.g. 3‑of‑5)
- Use https://safe.global
- Hardware wallets strongly recommended

## 2. Deploy Core Contracts
Deploy in this order:
1. `DistributorRegistry`
2. `BbsTreasury`
3. `BbsToken`

## 3. Wire Contracts
- Call `DistributorRegistry.setTreasury(treasuryAddress)`
- Call `BbsTreasury.setToken(tokenAddress)`
- Grant `MINTER_ROLE` on `BbsToken` to Treasury

## 4. Verify Contracts
- Verify all contracts on the block explorer
- Enables readable interaction and transparency

## 5. Operational Readiness
At this point the Foundation can:
- Mint tokens to Treasury
- Deploy distributors for sysops
- Fund distributors

No sysop onboarding should happen before this step.