# After a Sysop Applies — Foundation Workflow

This document describes what the Foundation does **after** a sysop submits an application.

## 1. Review Application
Evaluate:
- BBS identity and longevity
- Community relevance
- Technical readiness
- Sysop wallet address

Approval is social, not automatic.

## 2. Deploy Distributor
If approved:
- Treasury deploys `BbsMerkleDistributor`
- Sysop is set as distributor owner
- Distributor registered in `DistributorRegistry`

Relevant function:
- `BbsTreasury.deployDistributor()`

## 3. Fund Distributor
Foundation decides initial allocation.

Steps:
- Mint tokens to Treasury if needed (`mintToTreasury`)
- Transfer tokens to distributor (`fundDistributor`)

## 4. Notify Sysop
Provide:
- Distributor contract address
- Chain information
- Basic operating expectations

## 5. Ongoing Relationship
Foundation may:
- Top‑up distributor
- Pause or disable distributor if necessary
- Rotate sysop keys if compromised (with consent)

All actions require multisig approval.