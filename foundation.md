Foundation Guide (BBS-Coin)

This document describes the minimal responsibilities of the BBS-Coin Foundation. It intentionally focuses on onboarding sysops and funding distributors, which are the only required operational tasks today.

Governance, DAO mechanics, and long-term policy are intentionally left lightweight and evolutionary.

⸻

1. Role of the Foundation

The Foundation exists to:
	•	Maintain neutral custody of mint authority
	•	Prevent any single BBS from controlling supply
	•	Onboard new BBSes fairly
	•	Fund distributors transparently

The Foundation does not:
	•	Run a BBS
	•	Allocate rewards to users
	•	Decide how sysops award points

⸻

2. Ownership & Control Model

Safe Ownership (Recommended)

The Foundation controls:
	•	BbsTreasury
	•	DistributorRegistry

via a Gnosis Safe on Polygon.

Current state:
	•	Single signer (bootstrap phase)
	•	Designed to expand to multi-signer

This is not centralization by design, but a temporary operational reality.

⸻

3. Onboarding a New Sysop (Priority #1)

Inputs from Sysop
	•	Sysop owner wallet (EOA or Safe)
	•	BBS identifier (domain or name)
	•	Optional metadata URL

Actions Taken by Foundation
	1.	Deploy distributor

BbsTreasury.deployDistributor(sysopOwner, bbsId, metadataURI)

	2.	Register distributor

DistributorRegistry.register(...)

	3.	Enable distributor

BbsTreasury.setDistributorEnabled(distributor, true)

The sysop now controls epoch publishing.

⸻

4. Funding Distributors (Priority #2)

Minting

Tokens are minted only to the Treasury:

BbsTreasury.mintToTreasury(amount)

Funding a Distributor

BbsTreasury.fundDistributor(distributor, amount)

Properties:
	•	All funding is explicit and auditable
	•	Lifetime funding is tracked on-chain
	•	Distributors cannot mint

⸻

5. Emergency Powers (Rare)

If a sysop key is compromised or a severe error occurs:

BbsTreasury.revokeFromDistributor(distributor, amount)

This should be:
	•	Rare
	•	Publicly explained
	•	Used only to protect users

⸻

6. Current Governance Reality

Today:
	•	Governance is human
	•	Decisions are social
	•	Execution is on-chain

Future options (non-binding):
	•	Multi-sig expansion
	•	Advisory councils
	•	Token-based signaling

No DAO is required for BBS-Coin to function.

⸻

Summary

The Foundation exists to make BBS-Coin boring, predictable, and fair.

If sysops can onboard easily and users can trust the system, the Foundation is doing its job.