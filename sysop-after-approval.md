# BBS Coin — Sysop Guide (After Approval)

Welcome aboard. If you’re reading this, your BBS has been approved to participate in **BBS Coin** and you now have a distributor contract on-chain.

This document is written for **sysops**, not blockchain engineers. You do not need to understand Solidity to operate successfully — but you *do* need to understand what responsibilities you now have.

---

## What You Have Now

After approval, the Foundation has created the following **on your behalf**:

- A **BbsMerkleDistributor** contract
- Registered in the **Distributor Registry**
- Funded (or ready to be funded) with BBS Tokens
- Owned by **your wallet address** (EOA or Safe)

You should have received:
- Your **Distributor Contract Address**
- The **network** (Polygon mainnet)
- A link to the **claim page**
- Confirmation that tokens are available (or will be shortly)

If you do not have these, stop here and contact the Foundation.

---

## Your Role as a Sysop

You are responsible for:

- Deciding **how users earn points**
- Translating points → token allocations
- Publishing **epochs** (snapshots of who gets what)
- Hosting or serving epoch artifacts
- Communicating claim info to users

You are **not** responsible for:
- Minting tokens
- Registry administration
- Other BBS distributors
- Foundation governance

---

## Mental Model (Important)

Think in **epochs**, not balances.

- Each epoch is a snapshot in time
- Each user can claim **once per epoch**
- Amounts are **entitlements**, not deltas
- Unclaimed tokens do *not* roll over automatically

If a user earns tokens in Epoch 5 and doesn’t claim:
- Those tokens remain claimable for Epoch 5
- They must also claim Epoch 6 separately

---

## Preparing an Epoch (The Core Task)

At distribution time, you generate a simple JSON file:

```json
[
  {
    "address": "0xUserWalletAddress",
    "amount": "50000000000000000000"
  }
]
```

Important:
- `address` is the **user’s wallet**
- `amount` is in **base units** (18 decimals)
- This file may contain **many users**

> Yes, amounts look big.  
> `50 tokens = 50 * 10^18 = 50000000000000000000`

This file represents **exactly** what users may claim for that epoch.

---

## Building the Epoch Artifacts

From the input JSON, you generate:

- `root.json` → Merkle root
- `proofs.json` → proofs per wallet
- (optionally) the original input file

These artifacts must be **preserved**.

If they are lost:
- Users cannot claim
- Epoch cannot be reconstructed
- Tokens are effectively frozen

Treat them like backups.

---

## Publishing the Epoch (On-Chain)

Once artifacts are ready:

1. Connect your wallet
2. Call `publishEpoch(root, uri)` on your distributor
3. This increments the epoch number permanently

Rules:
- Epoch numbers cannot be reused
- Roots cannot be replaced
- Publishing bad data is irreversible

**Always validate before publishing.**

---

## Where to Host Epoch Data

You may host epoch artifacts anywhere:

- Static web hosting
- GitHub Pages
- IPFS
- Your BBS server
- blockbra.in, futureland.today, etc.

The blockchain stores only:
- The Merkle root
- A URI pointer

The blockchain does **not** validate your hosting.

If users cannot fetch proofs, they cannot claim.

---

## User Claims (What Users Do)

Users claim by calling:

```
claim(epoch, amount, proof)
```

They may do this:
- Via a web page
- Via a relayer (gas sponsored)
- Via a custom BBS tool
- Via scripts

You do **not** need to process claims manually.

---

## Communicating With Users

We recommend notifying users **after** an epoch is published.

Your notification should include:
- Epoch number
- Token amount
- Claim URL
- Their wallet address
- Any deadline (if you impose one)

Example claim link:

```
https://blockbra.in/bbscoin/tx.html
?chain=137
&to=0xYourDistributor
&epoch=5
&amount=50000000000000000000
&proof=[...]
```

Do **not** email private keys.
Do **not** guess wallet addresses.

---

## Common Footguns (Read This)

❌ Publishing an epoch with the wrong wallet addresses  
❌ Publishing before funding the distributor  
❌ Losing `proofs.json`  
❌ Mixing human-readable tokens with base units  
❌ Assuming claims “auto-roll over”  

If you are unsure, **do not publish**.

---

## Best Practices

- Dry-run epoch builds before publishing
- Keep artifacts in version control
- Log every epoch you publish
- Keep a local archive
- Start small (1–2 users) for first epoch

---

## When to Contact the Foundation

Contact the Foundation if:
- You published a bad epoch
- You need additional funding
- You lost epoch artifacts
- Your sysop key is compromised
- You want to pause or shut down distributions

---

## Final Notes

This system is designed to be:
- Transparent
- Minimal
- Non-custodial
- BBS-friendly

You are not issuing money.
You are issuing **participation credits** backed by cryptography.

Welcome to the experiment.

— BBS Coin Foundation
