---
stepsCompleted:
  - step-01-validate-prerequisites
  - step-02-design-epics
  - step-03-create-stories
  - step-04-final-validation
inputDocuments:
  - c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md
  - c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md
---

# Stalabard - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for Stalabard, decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

FR-001: System SHALL allow platform access only to verified community members.
FR-002: System SHALL verify membership using DAO NFT badge ownership.
FR-003: Every verified member SHALL be able to act as both seller and buyer.
FR-004: Seller views SHALL only expose that seller's own listings and seller operations.
FR-005: System SHALL deny non-member access to protected routes and actions.
FR-010: Verified member SHALL create listing with title, description, category, price, media, and metadata.
FR-011: Member SHALL publish listing directly without pre-approval.
FR-012: DAO moderator SHALL intervene only when an issue is flagged or policy violation is detected.
FR-013: Intervention SHALL support actions such as warning, suspend, or unpublish with explicit rationale.
FR-014: All intervention actions SHALL be auditable and visible to the listing owner.
FR-020: Buyers SHALL see only currently published and non-suspended listings.
FR-021: Sellers SHALL manage only their own listings.
FR-022: Orders SHALL preserve buyer and seller attribution.
FR-030: Checkout SHALL allow ELURC-only payment for MVP.
FR-031: Checkout SHALL support Phantom as first wallet option for verified members.
FR-032: Platform SHALL NOT custody private keys or sign user transactions.
FR-033: Payment flow SHALL verify transaction outcome before order completion.
FR-034: Payment record SHALL store token address, wallet address, tx hash, amount, and network.
FR-035: If payment provider is not ELURC, checkout SHALL fail with explicit error.
FR-040: System SHALL log listing lifecycle transitions.
FR-041: System SHALL log DAO moderation actions with actor, timestamp, action, and rationale.
FR-042: System SHALL expose operational summaries for listings, issues, moderation actions, orders, and payment outcomes.

### NonFunctional Requirements

NFR-001: Enforce role-based authorization on all custom APIs.
NFR-002: Enforce server-side ownership checks for seller-scoped resources.
NFR-003: Keep secrets server-side only; avoid exposing sensitive credentials in public runtime variables.
NFR-004: Include request-level audit metadata for governance and payment operations.
NFR-005: Membership verification checks MUST be server-side enforced for protected routes.
NFR-010: Payment verification path MUST be idempotent.
NFR-011: Listing publish and moderation workflows MUST be retry-safe.
NFR-012: API responses for moderation actions SHOULD return deterministic state transitions.
NFR-020: P95 response time <= 600ms for primary read/list APIs under MVP load.
NFR-021: Listing and order views SHOULD paginate and filter efficiently.
NFR-030: Correlate logs by member_id, listing_id, issue_id, order_id, payment_session_id.
NFR-031: Emit structured events for listing published/updated/suspended and payment confirmed/failed.

### Additional Requirements

- Starter template: No mandatory starter template is specified in the architecture document.
- Architecture pattern requirement: Implement Medusa v2 mutation architecture (module -> workflow -> API route).
- Runtime requirement: PostgreSQL persistence for core and custom module data.
- Runtime recommendation: Redis for workflow coordination, idempotency support, and short-lived verification cache.
- Integration requirement: Blockchain RPC integration for NFT ownership checks and ELURC transaction verification.
- Custom module requirement: Build and register `membership`, `marketplace`, `paymentMeta`, and `audit` modules.
- Data modeling requirement: Use module-link patterns and `query.graph()` for cross-module reads.
- Access control requirement: Enforce auth at middleware and ownership/policy checks in workflows (never trust frontend for security boundaries).
- Workflow requirement: Implement membership verification, listing create/publish, issue reporting, moderation action, ELURC enforcement, and payment verify-and-complete flows.
- Workflow reliability requirement: Step-level idempotency plus retry-safe transitions (publish/moderation/payment paths).
- API requirement: Use Zod request validation, `AuthenticatedMedusaRequest<T>`, and route-method conventions.
- Payment requirement: Enforce ELURC-only provider/token checks and non-custodial signing through Phantom.
- Payment state requirement: Deterministic payment status responses with retry-safe polling and metadata persistence.
- Observability requirement: Structured logs with correlation identifiers and auditable governance/payment events.
- Performance requirement: Pagination and indexed filters for seller/listing/order/payment query paths.
- Deployment requirement: Preserve migration lineage across local -> staging -> production.
- Release quality requirement: Pass E2E and authorization regression suites before MVP release.

### FR Coverage Map

FR-001: Epic 1 - Members-only access enforcement for verified community users.
FR-002: Epic 1 - DAO NFT badge ownership verification for membership gating.
FR-003: Epic 1 - Unified member role supporting both buying and selling.
FR-004: Epic 1 - Seller-scope authorization boundary for own resources only.
FR-005: Epic 1 - Protected route/action denial for non-members.
FR-010: Epic 2 - Listing creation capability with required listing fields.
FR-011: Epic 2 - Direct listing publication without pre-approval.
FR-012: Epic 5 - Reactive DAO moderation trigger on flagged issue or violation.
FR-013: Epic 5 - Moderation action model (warning/suspend/unpublish) with rationale.
FR-014: Epic 5 - Visibility and auditability of intervention actions to listing owners.
FR-020: Epic 3 - Buyer view restriction to published and non-suspended listings.
FR-021: Epic 2 - Seller listing management constrained to owner resources.
FR-022: Epic 3 - Order model preserving buyer/seller attribution.
FR-030: Epic 4 - ELURC-only checkout enforcement in MVP.
FR-031: Epic 4 - Phantom-first wallet support for verified members.
FR-032: Epic 4 - Non-custodial flow (platform never signs or stores private keys).
FR-033: Epic 4 - Payment outcome verification prerequisite for order completion.
FR-034: Epic 4 - Payment metadata persistence (token, wallet, tx hash, amount, network).
FR-035: Epic 4 - Explicit checkout failure for non-ELURC provider attempts.
FR-040: Epic 2 - Listing lifecycle transition logging.
FR-041: Epic 5 - DAO moderation audit log with actor/time/action/rationale.
FR-042: Epic 5 - Operational summary exposure for listings/issues/moderation/orders/payments.

## Epic List

### Epic 1: Member Access and Identity Enforcement
Verified community members can securely access platform capabilities as both buyers and sellers under server-side membership gating and strict ownership controls.
**FRs covered:** FR-001, FR-002, FR-003, FR-004, FR-005

### Epic 2: Seller Listing Creation and Publication
Verified member sellers can create, manage, and directly publish their own listings with full lifecycle traceability.
**FRs covered:** FR-010, FR-011, FR-021, FR-040

### Epic 3: Buyer Marketplace Discovery and Order Attribution
Verified member buyers can browse eligible listings and complete buyer flows where order records preserve seller and buyer attribution.
**FRs covered:** FR-020, FR-022

### Epic 4: ELURC Non-Custodial Checkout (Phantom First)
Verified member buyers can complete checkout through ELURC-only, Phantom-first, non-custodial payment flows with deterministic verification before order completion.
**FRs covered:** FR-030, FR-031, FR-032, FR-033, FR-034, FR-035

### Epic 5: Issue Reporting, DAO Moderation, and Governance Visibility
Community members and DAO moderators can report, resolve, and audit governance issues with clear rationale and operational visibility.
**FRs covered:** FR-012, FR-013, FR-014, FR-041, FR-042

## Epic 1: Member Access and Identity Enforcement

Verified community members can securely access platform capabilities as both buyers and sellers under server-side membership gating and strict ownership controls.

### Story 1.1: Verify DAO Membership from Connected Wallet

As a community member,
I want the platform to verify my DAO NFT badge ownership server-side when I connect my wallet,
So that only verified members can access protected marketplace features.

**Requirements:** FR-001, FR-002, NFR-005

**Acceptance Criteria:**

**Given** a user connects a wallet address
**When** the membership verification workflow executes
**Then** DAO NFT ownership is verified server-side against configured governance values
**And** member identity status is persisted with verification timestamp.

### Story 1.2: Enforce Members-Only Access on Protected Routes

As a platform operator,
I want protected routes to reject non-members,
So that marketplace actions are limited to verified community members.

**Requirements:** FR-001, FR-005, NFR-001

**Acceptance Criteria:**

**Given** a non-verified or failed-verification user
**When** they call a protected member route
**Then** the API returns forbidden with a deterministic error code
**And** no mutation or protected read is performed.

### Story 1.3: Enable Dual Buyer/Seller Capability for Verified Members

As a verified member,
I want one identity to access both buyer and seller capabilities,
So that I can participate in both sides of the marketplace without role conflicts.

**Requirements:** FR-003

**Acceptance Criteria:**

**Given** a verified member profile
**When** the member calls buyer and seller entry routes
**Then** both access paths are authorized under the same member identity
**And** role checks do not require separate account contexts.

### Story 1.4: Enforce Seller Ownership Scope Guard

As a seller,
I want access to be restricted to my own seller resources,
So that no seller can read or mutate another seller's listings or seller operations.

**Requirements:** FR-004, NFR-002

**Acceptance Criteria:**

**Given** seller A requests seller B resources
**When** the ownership guard evaluates seller_member_profile_id
**Then** access is denied with forbidden response
**And** an authorization denial event is logged.

### Story 1.5: Deliver Access-Control Regression Test Suite

As a development team,
I want regression tests for membership gating and ownership boundaries,
So that security constraints remain enforced across changes.

**Requirements:** FR-001, FR-004, FR-005, NFR-001, NFR-002

**Acceptance Criteria:**

**Given** the test suite is executed
**When** scenarios include non-member access and cross-seller requests
**Then** all unauthorized cases fail with expected status codes
**And** authorized verified-member cases pass consistently.

## Epic 2: Seller Listing Creation and Publication

Verified member sellers can create, manage, and directly publish their own listings with full lifecycle traceability.

### Story 2.1: Implement Listing Entity and Seller Link Model

As a backend developer,
I want a listing data model linked to seller identity,
So that listing ownership and lifecycle state can be enforced at data level.

**Requirements:** FR-010, FR-021

**Acceptance Criteria:**

**Given** the marketplace module migration is applied
**When** listing schema is created
**Then** required fields (title, description, category, price, media, metadata, status, seller link) exist
**And** indexes support seller-scope and status queries.

### Story 2.2: Create Seller Listing Draft Endpoint and Workflow

As a verified seller,
I want to create a draft listing with complete required attributes,
So that I can prepare inventory before publication.

**Requirements:** FR-010, FR-021

**Acceptance Criteria:**

**Given** an authenticated verified seller submits listing payload
**When** create-listing workflow executes
**Then** a draft listing is created under that seller profile
**And** attempts with invalid payload fail validation deterministically.

### Story 2.3: Update Own Listing Endpoint with Ownership Enforcement

As a seller,
I want to edit only my own listings,
So that listing management respects ownership boundaries.

**Requirements:** FR-021, NFR-002

**Acceptance Criteria:**

**Given** a seller updates a listing they own
**When** update workflow runs
**Then** fields are updated and audit metadata is recorded
**And** updates against another seller listing are rejected.

### Story 2.4: Publish Listing Workflow with Lifecycle Logging

As a verified seller,
I want to publish my listing directly,
So that it becomes available to eligible buyers without pre-approval.

**Requirements:** FR-011, FR-040, NFR-011

**Acceptance Criteria:**

**Given** a seller-owned draft listing and active membership status
**When** publish-listing workflow executes
**Then** status transitions to published with published timestamp
**And** lifecycle event is logged in structured audit format.

### Story 2.5: List Own Seller Listings with Pagination and Filters

As a seller,
I want paginated and filterable access to my own listings,
So that I can efficiently manage inventory at scale.

**Requirements:** FR-021, NFR-021

**Acceptance Criteria:**

**Given** a seller requests own listings
**When** optional status/date filters and pagination are provided
**Then** only seller-owned records are returned in deterministic order
**And** response includes pagination metadata.

## Epic 3: Buyer Marketplace Discovery and Order Attribution

Verified member buyers can browse eligible listings and complete buyer flows where order records preserve seller and buyer attribution.

### Story 3.1: Expose Published Non-Suspended Marketplace Listings

As a verified buyer,
I want to browse only valid listings,
So that I see items currently available for purchase.

**Requirements:** FR-020

**Acceptance Criteria:**

**Given** listings exist across draft, published, and suspended states
**When** buyer marketplace list endpoint is queried
**Then** only published and non-suspended listings are returned
**And** hidden states are excluded from payload.

### Story 3.2: Persist Buyer and Seller Attribution on Order Creation

As a platform operator,
I want orders to preserve buyer and seller attribution,
So that settlement, reporting, and governance review are auditable.

**Requirements:** FR-022

**Acceptance Criteria:**

**Given** a buyer creates an order for a listing
**When** order workflow executes
**Then** order stores buyer member profile linkage and seller attribution
**And** attribution fields are retrievable through order APIs.

### Story 3.3: Provide Buyer Order View with Attribution Context

As a verified buyer,
I want to view my order status and attribution details,
So that I can track who sold the item and current payment/order state.

**Requirements:** FR-022, NFR-021

**Acceptance Criteria:**

**Given** a buyer requests order detail for own order
**When** order view endpoint responds
**Then** payload includes buyer/seller attribution and status fields
**And** access to other buyers' orders is denied.

## Epic 4: ELURC Non-Custodial Checkout (Phantom First)

Verified member buyers can complete checkout through ELURC-only, Phantom-first, non-custodial payment flows with deterministic verification before order completion.

### Story 4.1: Enforce ELURC-Only Provider at Checkout

As a platform operator,
I want checkout to reject non-ELURC provider attempts,
So that MVP payment policy is consistently enforced.

**Requirements:** FR-030, FR-035

**Acceptance Criteria:**

**Given** checkout provider or token does not match configured ELURC values
**When** enforce-elurc-provider workflow executes
**Then** checkout fails fast with explicit policy error
**And** no payment session is marked eligible.

### Story 4.2: Integrate Phantom-First Non-Custodial Payment Initiation

As a verified buyer,
I want to sign payment transactions in Phantom,
So that the platform never handles my private keys.

**Requirements:** FR-031, FR-032

**Acceptance Criteria:**

**Given** a verified buyer proceeds to payment
**When** checkout payment initiation starts
**Then** Phantom is presented as the first supported wallet path
**And** backend does not request, store, or use any private key material.

### Story 4.3: Verify On-Chain Payment and Complete Order Idempotently

As a platform operator,
I want payment verification before order completion,
So that only valid confirmed ELURC transactions complete orders.

**Requirements:** FR-033, NFR-010

**Acceptance Criteria:**

**Given** a tx hash and payment session id are submitted
**When** verify-payment-and-complete-order workflow runs
**Then** token, amount, network, recipient, and confirmation policy are validated
**And** duplicate retries for same key return the same final verification result.

### Story 4.4: Persist Payment Metadata for Governance and Support

As a DAO moderator,
I want complete payment metadata persisted,
So that payment outcomes can be audited and investigated.

**Requirements:** FR-034, NFR-030

**Acceptance Criteria:**

**Given** payment verification reaches terminal state
**When** payment metadata is persisted
**Then** token address, wallet address, tx hash, amount, network, provider, and verification status are stored
**And** records are queryable by order and payment identifiers.

### Story 4.5: Expose Deterministic Payment Status Polling Endpoint

As a verified buyer,
I want deterministic payment status responses,
So that I can reliably track pending, failed, or confirmed outcomes.

**Requirements:** FR-033, NFR-010

**Acceptance Criteria:**

**Given** buyer polls payment status with order identifier
**When** backend evaluates payment state
**Then** response returns deterministic state (pending/confirmed/failed) and reason metadata
**And** repeated polling does not create duplicate side effects.

## Epic 5: Issue Reporting, DAO Moderation, and Governance Visibility

Community members and DAO moderators can report, resolve, and audit governance issues with clear rationale and operational visibility.

### Story 5.1: Enable Member Listing Issue Reporting

As a verified member,
I want to report problematic listings,
So that DAO moderators can review and act on potential policy violations.

**Requirements:** FR-012

**Acceptance Criteria:**

**Given** a verified member submits issue details for a listing
**When** report-listing-issue workflow executes
**Then** a listing issue record is created with reporter attribution and initial status
**And** a moderation queue event is emitted.

### Story 5.2: Provide Moderation Queue for DAO Moderators

As a DAO moderator,
I want a queue of flagged issues,
So that I can review pending governance actions efficiently.

**Requirements:** FR-012, NFR-021

**Acceptance Criteria:**

**Given** an authenticated moderator requests the moderation queue
**When** queue endpoint is called
**Then** flagged issues are returned with pagination and filtering
**And** non-moderator callers are denied.

### Story 5.3: Apply Moderation Actions with Rationale and State Transition Rules

As a DAO moderator,
I want to apply warning, suspend, or unpublish decisions with rationale,
So that interventions are explicit, deterministic, and enforce policy outcomes.

**Requirements:** FR-013, NFR-012

**Acceptance Criteria:**

**Given** a flagged issue in actionable status
**When** moderator submits action and rationale
**Then** moderation action record is created with actor, timestamps, previous/new status
**And** listing status transition follows deterministic workflow rules.

### Story 5.4: Expose Moderation History to Listing Owners

As a listing owner,
I want to view interventions applied to my listings,
So that I can understand moderation outcomes and rationale.

**Requirements:** FR-014

**Acceptance Criteria:**

**Given** listing owner requests moderation history for own listing
**When** history endpoint responds
**Then** actions include rationale, actor role, and status changes
**And** owners cannot access moderation history for listings they do not own.

### Story 5.5: Implement Governance and Operations Audit/Event Summaries

As a platform operator,
I want operational summaries for listings, issues, moderation, orders, and payments,
So that governance and reliability can be monitored continuously.

**Requirements:** FR-041, FR-042, NFR-004, NFR-030, NFR-031

**Acceptance Criteria:**

**Given** lifecycle, moderation, and payment events are emitted
**When** operational summary endpoints are queried
**Then** responses include aggregated and filterable metrics for required domains
**And** audit/event payloads include correlation identifiers and actor context.
