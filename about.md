BBS-Coin

1. Overview

What is BBS-Coin?

BBS-Coin is an ERC-20 token system designed to incentivize participation in Bulletin Board Systems (BBSes). It is explicitly not marketed as a financial instrument or investment. Tokens represent points, reputation, participation, or governance weight depending on how a community chooses to use them.

Key properties:
	•	ERC-20 compliant token (OpenZeppelin-based)
	•	Minting controlled by a Treasury contract
	•	Treasury is intended to be owned by a Safe (multisig)
	•	Multiple independent BBSes supported via per-BBS distributor contracts
	•	No single BBS controls the supply

ELI5

Think of BBS-Coin like arcade tickets:
	•	You earn tickets for doing stuff on a BBS
	•	Tickets can later be redeemed for prizes, votes, or bragging rights
	•	A neutral foundation prints the tickets
	•	Each arcade (BBS) hands them out independently

Architecture

BbsToken (ERC-20)
     ↑ mint (mintToTreasury)
BbsTreasury (Safe-owned)
     ↓ fund (fundDistributor)
BbsMerkleDistributor (per BBS)
     ↓ claim (claim / claimFor)
Users

Core Contracts

Contract	Purpose
BbsToken	ERC-20 token
BbsTreasury	Minting + distributor funding
DistributorRegistry	Discovery + metadata
BbsMerkleDistributor	Per-BBS reward distribution


⸻

2. Sysop Guide (How to Integrate a BBS)

What a Sysop Controls

A sysop:
	•	Owns a single BbsMerkleDistributor
	•	Decides how users earn points (posts, games, logins, etc.)
	•	Tracks those points off-chain
	•	Periodically converts points into Merkle epochs

A sysop does not:
	•	Mint tokens
	•	Control total supply
	•	Modify other BBSes

Distributor Lifecycle (with code references)
	1.	Distributor is deployed by the Foundation
→ BbsTreasury.deployDistributor(sysopOwner, bbsId, metadataURI)
	2.	Distributor is registered for discovery
→ DistributorRegistry.register(...) (called internally by Treasury)
	3.	Distributor is funded with tokens
→ BbsTreasury.fundDistributor(distributor, amount)
	4.	Sysop publishes an epoch
→ BbsMerkleDistributor.publishEpoch(bytes32 root, string uri)
	5.	Users claim tokens
→ BbsMerkleDistributor.claim(epoch, amount, proof)
→ or gas-sponsored: claimFor(account, epoch, amount, proof)

Epoch Model
	•	Epochs are sequential: 0, 1, 2, ...
	•	Each epoch is independent
	•	One claim per user per epoch
	•	Enforced on-chain via _claimed[epoch][account]

Merkle Leaf Definition (authoritative)

keccak256(abi.encodePacked(account, amount))

Ethers v6 equivalent:

keccak256(solidityPacked(["address","uint256"],[account, amountWei]))

Required Epoch Artifacts (must be preserved)

For each epoch, sysop tooling must persist:
	•	epoch.json – input snapshot (addresses + amounts)
	•	root.json – Merkle root published on-chain
	•	proofs.json – per-address { amount, leaf, proof[] }

If these artifacts are lost, users may be unable to claim even though the root exists on-chain.

⸻

3. Foundation / Treasury Operations

What the Foundation Controls

The Foundation (via Safe) is the only authority that:
	•	Mints new tokens
→ BbsTreasury.mintToTreasury(amount)
	•	Funds distributors
→ BbsTreasury.fundDistributor(distributor, amount)
	•	Registers distributors
→ DistributorRegistry.register(...)
	•	Enables / disables distributors
→ BbsTreasury.setDistributorEnabled(distributor, enabled)
	•	Recovers funds in emergencies
→ BbsTreasury.revokeFromDistributor(...)

Token Setup
	•	Token is deployed first: BbsToken
	•	Treasury address is granted MINTER_ROLE
	•	Token address is set once via:

BbsTreasury.setToken(tokenAddress)

Safe Ownership

Recommended:
	•	Polygon Safe
	•	2-of-3 or 3-of-5 signers
	•	Safe is owner of:
	•	BbsTreasury
	•	DistributorRegistry

⸻

4. Long-Term Vision

Design Principles
	•	Minimal trust
	•	Clear separation of concerns
	•	No forced token value
	•	No DAO theater

Possible Future Extensions
	•	claimMany() helper for UX
	•	Voting / governance using token balances
	•	Cross-BBS events
	•	Additional distributor tooling

Guiding Rule

BBS-Coin exists to make BBSes more fun — not to turn them into casinos.

⸻

Further Reading
	•	docs/sysop.md – detailed BBS integration & tooling
	•	docs/foundation.md – Safe operations & governance