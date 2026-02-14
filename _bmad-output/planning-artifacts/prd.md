---
stepsCompleted:
  - step-01-init
  - step-02-discovery
  - step-03-success
  - step-04-journeys
  - step-05-domain
  - step-06-innovation
  - step-07-project-type
  - step-08-scoping
  - step-09-functional
  - step-10-nonfunctional
  - step-11-polish
  - step-12-complete
inputDocuments:
  - c:/Projects/Stalabard/_bmad-output/planning-artifacts/product-brief-Stalabard-2026-02-14.md
  - c:/Projects/Stalabard/.env
workflowType: 'prd'
date: '2026-02-14'
classification:
  domain: general
  projectType: blockchain_web3
documentCounts:
  briefCount: 1
  researchCount: 0
  brainstormingCount: 0
  projectDocsCount: 0
---

# Product Requirements Document - Stalabard

**Author:** Thomas  
**Date:** 2026-02-14

## Executive Summary

Stalabard is a members-only Medusa.js marketplace for the Elurc DAO ecosystem. Verified members (DAO NFT badge holders) can both sell and buy, with direct listing publication enabled for MVP.

The MVP enforces server-side membership verification, strict seller ownership boundaries, reactive DAO moderation for flagged issues, and ELURC-only non-custodial checkout with Phantom as the first wallet.

## 1) Product Context

Stalabard is a Medusa.js marketplace for the Elurc DAO ecosystem. Access is members-only: people holding the DAO NFT badge can use the platform. Every verified member can both sell and buy.

### 1.1 Objectives

- Enable verified members to create and publish listings directly.
- Let DAO moderators intervene only when an issue is reported or detected.
- Restrict all platform usage to verified community members.
- Ensure sellers only access and manage their own resources.
- Support ELURC-only checkout with non-custodial wallets.
- Start wallet support with Phantom, then extend later.

### 1.2 Constraints

- Payment method must be ELURC only for MVP.
- Wallet model must be non-custodial (no private key custody by platform).
- Membership must be established through DAO NFT badge ownership.
- Only verified members can access platform features.
- Sellers must be scoped to their own listings and seller operations.
- DAO governance scope and membership proofs are tied to:
  - DAO address: `D6d8TZrNFwg3QG97JLLfTjgdJequ84wMhQ8f12UW56Rq`
  - NFT collection: `3e22667e998143beef529eda8c84ee394a838aa705716c3b6fe0b3d5f913ac4c`

## 2) Personas and Key Jobs

### 2.1 Community Member (Seller + Buyer)

- Can create and publish listings for sale.
- Can buy published listings from other members.
- Manages only own seller resources (listings and own seller-side order operations).
- Tracks listing status and payment settlement status.

### 2.2 DAO Moderator

- Intervenes on flagged issues or policy violations.
- Can suspend/unpublish listings with policy rationale.
- Needs auditable history.

### 2.3 Platform Operator (Secondary)

- Maintains system configuration and monitoring.
- Does not bypass membership and governance rules.

## 3) Core User Journeys

### Journey A: Member Selling Flow (Direct Listing)

1. Verified member connects Phantom wallet and passes membership verification.
2. Member creates listing details.
3. Member publishes listing directly.
4. Listing is available to other verified members.

### Journey B: Member Buying Flow (ELURC Checkout)

1. Verified member connects Phantom wallet.
2. Platform verifies member status (NFT badge ownership).
3. Member buyer browses published listings and adds items to cart.
4. System creates ELURC payment session.
5. Member buyer signs and broadcasts payment transaction.
6. Platform verifies transaction result and completes order.

### Journey C: Seller Ownership Scope

1. Seller views own listings/orders in seller views.
2. Seller attempts to access another seller's resources -> request is denied.
3. DAO moderator intervenes only for flagged issues across DAO governance scope.

## 4) Domain Model (MVP)

### 4.1 New/Custom Entities

- **MemberIdentity**: membership verification state and wallet linkage.
- **MemberProfile**: user-facing profile and member metadata.
- **Listing**: sellable item created and published directly by a verified member.
- **ListingIssue**: reported issue attached to a listing and moderation status.
- **ModerationAction**: DAO intervention action and rationale.
- **PaymentIntentMetadata**: tx hash, chain/network, wallet address, token amount.

### 4.2 Key Relationships

- MemberIdentity 1..1 MemberProfile
- MemberProfile 1..* Listing (seller ownership)
- Listing 1..* ListingIssue
- ListingIssue 1..* ModerationAction
- MemberProfile 1..* Order (buyer side)
- Order 1..1 PaymentIntentMetadata

## 5) Functional Requirements

### 5.1 Membership and Access Control

- **FR-001**: System SHALL allow platform access only to verified community members.
- **FR-002**: System SHALL verify membership using DAO NFT badge ownership.
- **FR-003**: Every verified member SHALL be able to act as both seller and buyer.
- **FR-004**: Seller views SHALL only expose that seller's own listings and seller operations.
- **FR-005**: System SHALL deny non-member access to protected routes and actions.

### 5.2 Listing Publication and DAO Intervention

- **FR-010**: Verified member SHALL create listing with title, description, category, price, media, and metadata.
- **FR-011**: Member SHALL publish listing directly without pre-approval.
- **FR-012**: DAO moderator SHALL intervene only when an issue is flagged or policy violation is detected.
- **FR-013**: Intervention SHALL support actions such as warning, suspend, or unpublish with explicit rationale.
- **FR-014**: All intervention actions SHALL be auditable and visible to the listing owner.

### 5.3 Listings and Orders

- **FR-020**: Buyers SHALL see only currently published and non-suspended listings.
- **FR-021**: Sellers SHALL manage only their own listings.
- **FR-022**: Orders SHALL preserve buyer and seller attribution.

### 5.4 ELURC Payment (Non-Custodial)

- **FR-030**: Checkout SHALL allow ELURC-only payment for MVP.
- **FR-031**: Checkout SHALL support Phantom as first wallet option for verified members.
- **FR-032**: Platform SHALL NOT custody private keys or sign user transactions.
- **FR-033**: Payment flow SHALL verify transaction outcome before order completion.
- **FR-034**: Payment record SHALL store token address, wallet address, tx hash, amount, and network.
- **FR-035**: If payment provider is not ELURC, checkout SHALL fail with explicit error.

### 5.5 Auditability and Admin

- **FR-040**: System SHALL log listing lifecycle transitions.
- **FR-041**: System SHALL log DAO moderation actions with actor, timestamp, action, and rationale.
- **FR-042**: System SHALL expose operational summaries for listings, issues, moderation actions, orders, and payment outcomes.

## 6) Non-Functional Requirements

### 6.1 Security

- **NFR-001**: Enforce role-based authorization on all custom APIs.
- **NFR-002**: Enforce server-side ownership checks for seller-scoped resources.
- **NFR-003**: Keep secrets server-side only; avoid exposing sensitive credentials in public runtime variables.
- **NFR-004**: Include request-level audit metadata for governance and payment operations.
- **NFR-005**: Membership verification checks MUST be server-side enforced for protected routes.

### 6.2 Reliability

- **NFR-010**: Payment verification path MUST be idempotent.
- **NFR-011**: Listing publish and moderation workflows MUST be retry-safe.
- **NFR-012**: API responses for moderation actions SHOULD return deterministic state transitions.

### 6.3 Performance

- **NFR-020**: P95 response time <= 600ms for primary read/list APIs under MVP load.
- **NFR-021**: Listing and order views SHOULD paginate and filter efficiently.

### 6.4 Observability

- **NFR-030**: Correlate logs by member_id, listing_id, issue_id, order_id, payment_session_id.
- **NFR-031**: Emit structured events for listing published/updated/suspended and payment confirmed/failed.

## 7) API/Workflow Requirements (Medusa-focused)

### 7.1 Custom Module Requirements

- Build membership module for member identity verification (NFT badge ownership checks).
- Build marketplace module for listing ownership, issues, and moderation actions.
- Define links between marketplace entities and Medusa Product/Order/Payment domains as needed.

### 7.2 Workflow Requirements

- Workflow to verify membership before protected actions.
- Workflow to create and publish listing.
- Workflow to report listing issue.
- Workflow to perform DAO moderation action on issue.
- Workflow/hook to enforce ELURC-only payment provider.
- Workflow/step to verify non-custodial payment result before order completion.

### 7.3 Route Requirements

- Member listing routes (create, update, publish, list-own).
- Issue routes (report issue, list own reports, get issue status).
- DAO moderation routes (list flagged issues, act on issue).
- Payment status route for checkout confirmation status.

## 8) MVP Scope and Release

### 8.1 In Scope (MVP)

- Members-only access based on DAO NFT badge ownership.
- Member-driven direct listing publication.
- DAO intervention flow for issues only.
- Seller ownership scoping (own resources only).
- Phantom-first non-custodial checkout.
- ELURC-only provider enforcement.
- Basic audit/event logging.

### 8.2 Out of Scope (MVP)

- Additional wallet providers beyond Phantom.
- Multi-token or fiat rails.
- Full dispute/returns management automation.
- Advanced recommendation/personalization.

### 8.3 Release Readiness Criteria

- End-to-end scenario passes:
  membership verification -> listing publish -> phantom payment -> order completion.
- Seller ownership-scope test suite passes.
- Members-only access test suite passes.
- DAO issue-intervention test suite passes.
- ELURC-only enforcement tests pass.
- Audit/event trail available for governance and payment events.

## 9) Acceptance Criteria (System Level)

1. Given a non-member wallet, when attempting platform actions, then access is denied.
2. Given a verified member, when creating and publishing a listing, then listing is visible to other verified members.
3. Given seller A, when requesting seller B's listings/orders in seller scope, then API returns forbidden.
4. Given a flagged listing issue, DAO moderator can apply intervention action with rationale.
5. Given a verified member buyer, when provider is ELURC and payment is confirmed, then order completes and metadata is persisted.
6. Given moderation action, when decision is recorded, then audit history includes actor, timestamp, previous status, new status, and rationale.

## 10) Risks and Mitigations

- **Risk**: Membership verification latency or reliability issues.
  - **Mitigation**: deterministic verification step with clear failure modes and retries.
- **Risk**: Harmful listings remain live before moderation.
  - **Mitigation**: issue reporting, rapid moderator intervention tools, and clear policy rules.
- **Risk**: Ownership leaks through custom query paths.
  - **Mitigation**: centralized ownership guard and integration tests.
- **Risk**: On-chain confirmation delays degrade UX.
  - **Mitigation**: explicit pending state and retry-safe status polling.

## 11) Open Decisions (to finalize during architecture)

1. Exact membership verification mechanism and cache policy for NFT ownership checks.
2. Intervention policy model (manual moderation only vs configurable policy automation later).
3. Exact payment verification strategy (RPC confirmation depth, timeout, and reconciliation behavior).

## 12) Traceability Matrix

- Members-only access -> FR-001, FR-002, FR-005, NFR-005
- Every member can sell and buy -> FR-003, Journey A, Journey B
- Seller sees own resources only -> FR-004, FR-021, Acceptance Criteria #3
- DAO issue intervention flow -> FR-010..FR-014, FR-040..FR-041
- ELURC-only + Phantom-first non-custodial -> FR-030..FR-035, NFR-010
