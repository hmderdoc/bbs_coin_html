# Foundation Chain Operations (Multisig / Safe Runbook)

This document is **the practical, technical guide** for how the BBS Coin Foundation
actually operates contracts on-chain.

No DAO theory, no governance hype — just how adults run smart contracts with a multisig.

---

## 1. What “the Foundation” is on-chain

On-chain, **the Foundation is the Safe**.

There is no special Foundation contract, role, or magic address.
Whoever can execute transactions **from the Safe** controls Foundation authority.

Key implication:
- If the Safe owns `BbsTreasury`, then **only Safe-approved transactions** can:
  - mint tokens
  - deploy distributors
  - fund distributors
  - enable / disable distributors
  - perform emergency actions

Everything else is off-chain process and human coordination.

---

## 2. Mental model: how multisig execution actually works

A Safe transaction has four real stages:

1. **Propose**
   - One signer creates a transaction
   - Defines `to`, `value`, and `data`
2. **Review**
   - Other signers inspect the transaction details
3. **Approve**
   - Signers approve until quorum is met
4. **Execute**
   - One signer submits the transaction on-chain

Important:
- Creating a transaction ≠ executing it
- Approving ≠ executing
- Nothing happens on-chain until execution

---

## 3. What the Foundation actually does (and does not do)

### Foundation DOES
- Own and operate `BbsTreasury`
- Mint tokens into Treasury
- Deploy distributor contracts
- Fund distributor contracts
- Enable / disable distributors
- Recover funds in emergencies

### Foundation DOES NOT
- Publish Merkle epochs
- Generate Merkle trees
- Handle user claims
- Manage BBS integrations
- Decide how sysops calculate rewards

Those responsibilities belong to sysops and their tooling.

---

## 4. Transactions vs call data (critical distinction)

A **transaction** is:
- `to` address
- `value` (usually 0)
- `data` (ABI-encoded function call)

**Call data** is just the encoded function call.
Safe executes transactions — not “functions”.

You must always know:
- what contract you are calling
- which function
- with what parameters
- what state change that causes

---

## 5. How the Foundation generates call data

There are two supported paths.

### Option A: Safe UI (simplest)
Use Safe’s **Contract Interaction** feature:
- Paste or select contract ABI
- Choose function
- Enter parameters
- Safe builds the call data

Pros:
- No scripting
- Harder to screw up encoding

Cons:
- Manual
- Slower at scale

---

### Option B: Generate call data with scripts (recommended long-term)

Use `ethers` to generate call data locally, then paste into Safe.

Example pattern:
```ts
import { ethers } from "ethers";

const abi = [
  "function fundDistributor(address distributor, uint256 amount)"
];

const iface = new ethers.Interface(abi);
const data = iface.encodeFunctionData(
  "fundDistributor",
  ["0xDistributor", 500n * 10n**18n]
);

console.log(data);
```

Safe inputs:
- to: `BbsTreasury` address
- value: `0`
- data: output from script

Pros:
- Auditable
- Repeatable
- Commit scripts to Git

---

## 6. Concrete Foundation operations (tied to your contracts)

Below are **real actions** the Foundation will take, mapped directly to Solidity.

---

### 6.1 Set token on Treasury (one-time)

Contract:
- `BbsTreasury`

Function:
```solidity
function setToken(address token_)
```

When:
- After deploying `BbsToken`

Safe transaction:
- to: `BbsTreasury`
- value: `0`
- data: `setToken(tokenAddress)`

Failure conditions:
- Reverts if already set

---

### 6.2 Grant MINTER_ROLE to Treasury (one-time)

Contract:
- `BbsToken`

Functions:
```solidity
function MINTER_ROLE() external view returns (bytes32)
function grantRole(bytes32 role, address account)
```

When:
- After Treasury deployment
- Before minting

Effect:
- Allows Treasury to mint tokens

---

### 6.3 Mint tokens to Treasury

Contract:
- `BbsTreasury`

Function:
```solidity
function mintToTreasury(uint256 amount)
```

When:
- Treasury needs supply to fund distributors

Important:
- `amount` is in base units (18 decimals)

---

### 6.4 Deploy a distributor (sysop onboarding)

Contract:
- `BbsTreasury`

Function:
```solidity
function deployDistributor(
  address sysopOwner,
  string bbsId,
  string metadataURI
)
```

What this does:
- Deploys `BbsMerkleDistributor`
- Registers it in `DistributorRegistry`
- Enables it for funding

Output:
- Distributor address emitted in `DistributorDeployed` event

---

### 6.5 Fund a distributor

Contract:
- `BbsTreasury`

Function:
```solidity
function fundDistributor(address distributor, uint256 amount)
```

Checks:
- Distributor must be enabled
- Treasury must have sufficient balance

---

### 6.6 Disable or re-enable a distributor

Contract:
- `BbsTreasury`

Function:
```solidity
function setDistributorEnabled(address distributor, bool enabled)
```

Use cases:
- Sysop key compromised
- Abuse
- Temporary suspension

---

### 6.7 Emergency recovery (optional)

Contract:
- `BbsTreasury` → `BbsMerkleDistributor`

Function chain:
```solidity
BbsTreasury.revokeFromDistributor(...)
→ BbsMerkleDistributor.revokeToTreasury(...)
```

Purpose:
- Pull unclaimed tokens back to Treasury

---

## 7. Safe execution checklist (do this every time)

Before approving a Safe transaction:

- [ ] Correct chain (Polygon vs testnet)
- [ ] `to` address matches canonical contract
- [ ] `value == 0`
- [ ] Function name and params correct
- [ ] Amounts use base units (18 decimals)
- [ ] Action matches Foundation intent
- [ ] Sysop address verified (for deploys)

If any box is unchecked, do not approve.

---

## 8. Separation of responsibilities (burn this in)

- **Foundation**
  - Controls supply
  - Controls distributors
  - Provides funding
- **Sysop**
  - Publishes epochs
  - Generates Merkle trees
  - Operates BBS integration
- **User**
  - Claims tokens
  - Owns wallet

If these blur, bugs and blame follow.

---

## 9. Operational record keeping (not optional)

Maintain a simple log per chain:
- Date
- Safe tx ID
- On-chain tx hash
- Operation type
- Parameters
- Link to explorer
- Notes

This matters more than clever tooling.

---

## 10. Final principle

The Safe is not a convenience.
It is the *constitution* of the Foundation.

If it’s not executed by the Safe,
it didn’t happen.
