# Sysop Guide (BBS-Coin)

This document explains **how a BBS sysop joins BBS-Coin, prepares reward data,
and publishes epochs**.

It is intentionally practical and minimal.
If you run a BBS and want to distribute BBS-Coin to your users, this is the only
document you need.

---

## 0. Mental Model (Read This First)

- You **do not mint tokens**
- You **do not control supply**
- You are given a **Distributor contract** that already holds tokens
- Your responsibility is limited to:
  1. Deciding who earns tokens
  2. Publishing that decision as an immutable snapshot (epoch)
  3. Letting users claim

Everything else is off-chain bookkeeping.

---

## 1. Signup & Onboarding (Sysop ↔ Foundation)

### What the Sysop Provides

- Sysop wallet address (EOA or Safe)
- BBS identifier (e.g. `futureland.today`)
- Optional metadata URL

### What the Foundation Does

1. Deploys your distributor  
   `BbsTreasury.deployDistributor(sysopOwner, bbsId, metadataURI)`

2. Registers it  
   `DistributorRegistry.register(...)`

3. Funds it  
   `BbsTreasury.fundDistributor(distributor, amount)`

You receive a distributor address that is ready to use once funded.

---

## 2. Preparing Reward Data

At distribution time, produce:

```json
[
  { "address": "0xUser1", "amount": "50000000000000000000" }
]
```

Rules:
- `amount` is base units (wei) BBS Token uses 18 decimals, so:
50 tokens = 50 × 10¹⁸ = 50000000000000000000
- One entry per address per epoch
- Amount is the total entitlement for that epoch

---

## 3. Leaf Hash (Authoritative)

Solidity:

```solidity
keccak256(abi.encodePacked(account, amount))
```

Ethers v6:

```ts
ethers.keccak256(
  ethers.solidityPacked(["address","uint256"], [account, amount])
)
```

Any deviation breaks claims permanently.

---

## 4. Required Epoch Artifacts

You **must** persist:

- `epoch.json`
- `root.json`
- `proofs.json`

Loss of these may prevent claims.

---

## 5. Publishing the Epoch

```solidity
publishEpoch(root, uri)
```

Epochs are:
- Sequential
- Immutable
- Permanent

---

## 6. Claims

Users claim per epoch:

```solidity
claim(epoch, amount, proof)
```

Unclaimed epochs do not expire.

---

## 7. Common Mistakes

- Wrong leaf hash
- Publishing without saving artifacts
- Assuming epochs can be corrected

---

## Summary

Publish only what you are willing to make permanent.
