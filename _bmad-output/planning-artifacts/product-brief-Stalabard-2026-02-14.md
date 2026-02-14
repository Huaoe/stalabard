---
stepsCompleted:
  - step-01-init
  - step-02-vision
  - step-03-users
  - step-04-metrics
  - step-05-scope
  - step-06-complete
inputDocuments:
  - c:/Projects/Stalabard/.env
  - user-conversation-context
date: 2026-02-14
author: Thomas
---

# Product Brief: Stalabard

## Executive Summary

Stalabard is a multi-tenant commerce platform for the Elurc community governed by its DAO. The product enables sub-organizations (for example solo shops, grocery stores, and future vertical communities) to propose items, receive governance approval, and sell approved items in a shared marketplace infrastructure.

The platform is built around three non-negotiable principles:

1. **Community governance first**: item listing begins as a proposal and follows a review flow aligned to DAO policy.
2. **Tenant autonomy with shared infrastructure**: each organization manages its own catalog and operations without crossing data boundaries.
3. **ELURC-native commerce**: checkout supports **ELURC token-only** payments with **non-custodial wallets**, using **Phantom as the first wallet integration**.
4. **Members-only access**: platform usage is restricted to community members coopted with BrightID and holding the community NFT badge.

---

## Core Vision

### Problem Statement

The Elurc ecosystem needs a trusted, scalable way for many community-run organizations to sell goods under one platform while preserving governance, transparency, and economic alignment with ELURC.

### Problem Impact

Without a multi-tenant DAO-governed commerce platform:

- Organizations rely on fragmented storefronts and inconsistent processes.
- Governance of item quality and legitimacy is difficult to enforce.
- Treasury alignment weakens when payment rails are not ELURC-native.
- Community trust degrades when seller boundaries and approvals are opaque.

### Why Existing Solutions Fall Short

- Traditional marketplaces are not DAO-native and don’t model proposal/approval governance well.
- Typical ecommerce stacks assume fiat/credit-card-first payments.
- Generic multitenant systems often under-implement role boundaries and auditability for community governance.

### Proposed Solution

Build a Medusa-based multi-tenant backend and composable storefront where:

- Community membership is established through **BrightID cooptation**.
- Coopted people receive an **NFT badge** that proves they are community members.
- Only NFT-badge members can access and use platform features (proposal, review, and purchase based on role).
- Organizations submit item proposals.
- DAO reviewers approve/reject proposals via auditable workflow.
- Approved proposals convert into sellable catalog entries scoped to the proposing organization.
- Buyers connect non-custodial wallets and pay in ELURC only.
- Admin operations and reporting are tenant-aware by default.

### Key Differentiators

- DAO-governed listing pipeline rather than direct publish.
- Members-only access model based on BrightID cooptation + NFT badge ownership.
- ELURC token-only checkout and treasury alignment.
- Phantom-first non-custodial UX while preserving future wallet extensibility.
- Tenant-isolated operations in one shared Medusa application.

## Target Users

### Primary Users

1. **Organization Admin (Seller Operator)**
   - Runs a sub-organization store (solo shop, grocery, niche shop).
   - Needs simple proposal submission, inventory control, and order handling.
   - Success = can onboard items and fulfill orders without governance confusion.

2. **DAO Reviewer / Governance Operator**
   - Reviews item proposals against community standards.
   - Needs queue visibility, policy checks, and clear decision trail.
   - Success = can approve/reject quickly and transparently.

3. **Community Buyer (Member Buyer)**
   - Purchases trusted products from approved organizations.
   - Uses non-custodial wallet (Phantom first).
   - Success = smooth wallet connect, clear ELURC pricing, reliable checkout.

### Secondary Users

- **Treasury/Finance Steward**: validates ELURC payment settlement behavior and reporting.
- **Platform Operator**: manages regions, channels, and technical operations.

### User Journey

1. Organization admin signs in and creates an item proposal.
2. DAO reviewer evaluates and approves/rejects with rationale.
3. Approved item becomes active listing under the organization.
4. Buyer discovers item, connects Phantom wallet, pays in ELURC.
5. Organization processes order; governance and operations remain auditable.

## Success Metrics

- Proposal-to-decision median time.
- Proposal approval rate (with category breakdown).
- Approved proposal to first-sale conversion rate.
- Checkout completion rate for ELURC wallet flow.
- Failed payment-session rate.
- Tenant boundary incident count (target: zero).

### Business Objectives

- Launch a reliable DAO-governed marketplace core.
- Increase ELURC economic utility through actual commerce volume.
- Enable multiple organization types to operate independently on shared infrastructure.

### Key Performance Indicators

- **KPI-1**: >= 90% of proposals receive DAO decision within 72 hours.
- **KPI-2**: >= 95% successful wallet-based checkout after payment session creation.
- **KPI-3**: 0 cross-tenant data access incidents.
- **KPI-4**: >= 30% of approved items receive at least one order within 30 days.

## MVP Scope

### Core Features

1. Organization onboarding and role model (owner/manager/proposer/reviewer).
2. Proposal lifecycle (draft, submitted, approved, rejected).
3. Conversion of approved proposals to sellable listings.
4. Tenant-scoped catalog and order management.
5. ELURC-only checkout through custom Medusa payment provider.
6. Non-custodial wallet integration with Phantom as first supported wallet.
7. Basic audit logs for proposal decisions and payment outcomes.

### Out of Scope for MVP

- Multi-chain token support beyond ELURC environment.
- Advanced recommendation/search ranking.
- Complex dispute/returns automation.
- Deep analytics warehouse integrations.

### MVP Success Criteria

- End-to-end flow works in production-like environment:
  proposal -> approval -> listing -> ELURC checkout -> order creation.
- Governance roles are enforceable and auditable.
- Wallet checkout UX is clear for first-time Phantom users.

### Future Vision

- Additional non-custodial wallets (Backpack, Solflare, WalletConnect adapters if needed).
- DAO-configurable policy engine for category-specific approval rules.
- Automated treasury reconciliation and on-chain/off-chain reporting.
- Reputation scoring for organizations and proposal quality.
