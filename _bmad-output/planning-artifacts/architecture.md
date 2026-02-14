---
stepsCompleted:
  - 1
  - 2
inputDocuments:
  - c:/Projects/Stalabard/_bmad-output/planning-artifacts/product-brief-Stalabard-2026-02-14.md
  - c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md
workflowType: 'architecture'
project_name: 'Stalabard'
user_name: 'Thomas'
date: '2026-02-14'
---

# Architecture Decision Document

_This document defines the target architecture for MVP implementation using Medusa v2 patterns (module -> workflow -> API route), based on the PRD and product brief._

## 1. Architecture Scope and Alignment

### 1.1 Source Alignment

This architecture is based on:

- `prd.md` as the primary implementation baseline for MVP behavior.
- `product-brief-Stalabard-2026-02-14.md` as strategic direction.

### 1.2 Functional Baseline for MVP

The MVP implementation baseline is:

1. Members-only marketplace (NFT badge ownership required server-side).
2. Every verified member can buy and sell.
3. Seller resources are ownership-scoped (no cross-seller access).
4. Listings are directly publishable by verified members.
5. DAO moderation is reactive (issue/policy violation driven).
6. ELURC-only checkout with non-custodial wallet flow (Phantom first).

### 1.3 Strategic Extensions from Product Brief

The product brief introduces future-ready direction:

- Multi-tenant organizations and governance role depth.
- Proposal -> DAO decision -> listing conversion pipeline.

For MVP, these are included as extensibility points without changing the PRD baseline flow.

---

## 2. High-Level Architecture

### 2.1 Layered Medusa Architecture

Stalabard uses the required Medusa mutation architecture:

1. **Modules**: data models + CRUD services.
2. **Workflows**: all mutation business logic, validation, compensation, idempotency.
3. **API routes**: HTTP interface, auth middleware, typed request validation.
4. **Storefront/Admin clients**: consume store/admin APIs via SDK.

### 2.2 Runtime Components

- **Medusa Backend (Node/TypeScript)**
  - Core Medusa commerce modules (product/order/cart/payment/customer/user).
  - Custom domain modules (membership + marketplace + payment metadata + audit).
- **PostgreSQL**
  - Persists Medusa core + custom module data.
- **Redis (recommended)**
  - Workflow coordination, idempotency support, short-lived verification cache.
- **Blockchain RPC integration layer**
  - NFT ownership checks and ELURC transaction verification.
- **Storefront (Next.js)**
  - Wallet connect UX (Phantom first) + buyer/seller screens.

---

## 3. Domain and Module Architecture

### 3.1 Custom Modules

#### A) `membership` module

Purpose: membership proof + wallet identity.

Core entities:

- `member_identity`
  - `id`, `wallet_address` (unique), `membership_status`, `verified_at`, `source`.
- `member_profile`
  - `id`, `member_identity_id`, `display_name`, `avatar_url`, `metadata`.

Notes:

- Membership checks are always enforced server-side before protected actions.
- Cache may be used for read performance, but cannot bypass server verification policy.

#### B) `marketplace` module

Purpose: seller-owned listings + issue moderation.

Core entities:

- `listing`
  - `id`, `seller_member_profile_id`, `status` (draft|published|suspended|unpublished),
    `title`, `description`, `category`, `price`, `media`, `metadata`, `published_at`.
- `listing_issue`
  - `id`, `listing_id`, `reporter_member_profile_id`, `status`, `reason`, `details`.
- `moderation_action`
  - `id`, `issue_id`, `listing_id`, `moderator_user_id`, `action`, `rationale`,
    `previous_status`, `new_status`, `acted_at`.

#### C) `paymentMeta` module

Purpose: non-custodial payment intent/verification metadata.

Core entities:

- `payment_intent_metadata`
  - `id`, `order_id`, `wallet_address`, `token_address`, `amount`, `network`,
    `tx_hash`, `provider`, `verification_status`, `confirmed_at`, `failure_reason`.

#### D) `audit` module

Purpose: immutable governance/payment audit events.

Core entity:

- `audit_event`
  - `id`, `event_type`, `actor_type`, `actor_id`, `subject_type`, `subject_id`,
    `correlation_id`, `payload`, `created_at`.

### 3.2 Module Naming and Structure

- Use camelCase module names only (`membership`, `marketplace`, `paymentMeta`, `audit`).
- Folder pattern:
  - `src/modules/<name>/models/*`
  - `src/modules/<name>/service.ts`
  - `src/modules/<name>/index.ts`

### 3.3 Module Links (Cross-Module)

Define links (instead of direct cross-module service coupling):

- `member_profile` -> Medusa customer/user (for actor mapping).
- `listing` -> Medusa product variant/product (sellable mapping).
- `listing_issue` -> `listing`.
- `moderation_action` -> `listing_issue` and `listing`.
- `payment_intent_metadata` -> Medusa order/payment collection.

Use `query.graph()` for cross-module reads and reporting.

---

## 4. Access Control Architecture

### 4.1 Authentication

- **Store routes**: authenticated member (session/bearer) + membership verification workflow.
- **Admin routes**: authenticated DAO moderator/platform user (session/bearer).

### 4.2 Authorization Model

- **Member role**: buyer + seller capabilities.
- **Seller scope rule**: seller can access only resources where
  `resource.seller_member_profile_id === requester.member_profile_id`.
- **Moderator role**: can act only on moderation routes and must provide rationale.

### 4.3 Enforcement Location

- Route middleware handles auth + request schema validation.
- Workflow steps enforce ownership and policy checks.
- No ownership/security logic is trusted to frontend.

---

## 5. Workflow Architecture (Mutation Flows)

All mutation operations use workflows. Key workflows:

1. `verify-membership-workflow`
   - Input: wallet/member context.
   - Validates NFT ownership for configured DAO collection.
   - Updates `member_identity` status.

2. `create-listing-workflow`
   - Validates member eligibility.
   - Creates seller-owned listing.

3. `publish-listing-workflow`
   - Validates owner + membership.
   - Moves listing to `published`.
   - Emits audit/event.

4. `report-listing-issue-workflow`
   - Creates issue with reporter attribution.
   - Emits moderation queue event.

5. `apply-moderation-action-workflow`
   - Moderator-only.
   - Validates deterministic state transition.
   - Writes `moderation_action` and updates listing status.
   - Emits audit/event.

6. `enforce-elurc-provider-workflow`
   - Validates payment provider/token == ELURC only.
   - Fails fast with explicit error when invalid.

7. `verify-payment-and-complete-order-workflow`
   - Idempotent by payment session/order + tx hash.
   - Verifies on-chain tx status/amount/token/recipient.
   - Persists `payment_intent_metadata`.
   - Completes order only after confirmation rules pass.

Workflow design constraints:

- Step-level idempotency required (payment and publish paths).
- Compensation where rollback is possible.
- Retry-safe transitions for listing and moderation actions.

---

## 6. API Architecture

### 6.1 Route Groups

#### Store/member routes

- `POST /store/member/listings` (create)
- `POST /store/member/listings/:id` (update using POST convention)
- `POST /store/member/listings/:id/publish`
- `GET /store/member/listings` (own listings)
- `GET /store/listings` (published non-suspended marketplace listings)
- `POST /store/member/issues` (report)
- `GET /store/member/issues` (own reports)
- `GET /store/member/issues/:id`
- `GET /store/member/payment-status/:order_id`

#### Admin/moderation routes

- `GET /admin/moderation/issues` (flagged queue)
- `POST /admin/moderation/issues/:id/actions`
- `GET /admin/moderation/actions`

### 6.2 API Standards

- Request body validation via Zod + `validateAndTransformBody` middleware.
- Use `AuthenticatedMedusaRequest<T>` for protected routes.
- Keep business logic in workflows, not in route handlers.
- Use Medusa route method conventions (GET/POST/DELETE).

---

## 7. Payment Architecture (ELURC, Non-Custodial)

### 7.1 Payment Principles

- Platform never handles private keys.
- Buyer signs transaction in Phantom wallet.
- Backend verifies transaction outcome before order completion.
- Non-ELURC payment attempts are rejected.

### 7.2 Payment Processing Sequence

1. Checkout creates payment session with custom ELURC provider context.
2. Storefront requests wallet signature/broadcast.
3. Tx hash returned to backend status endpoint.
4. Verification workflow validates:
   - token address == configured ELURC token,
   - amount and network match order/session,
   - transaction confirmed by configured confirmation policy.
5. If valid -> order completion + metadata persistence.
6. If invalid/pending -> deterministic status response + retry-safe polling.

### 7.3 Idempotency and Reconciliation

- Idempotency key scope: `payment_session_id + tx_hash`.
- Duplicate callbacks/polls return same final state.
- Reconciliation job (post-MVP optional) can re-check unresolved pending states.

---

## 8. Observability and Audit Architecture

### 8.1 Structured Logging

Every protected workflow and route emits structured logs with:

- `member_id`, `listing_id`, `issue_id`, `order_id`, `payment_session_id`, `correlation_id`.

### 8.2 Audit Events (Minimum)

- Listing created/updated/published/suspended/unpublished.
- Issue reported/status updated.
- Moderation decision recorded with rationale and actor.
- Payment confirmed/failed/pending timeout.

### 8.3 Operational Views

Expose summaries for:

- listing lifecycle,
- moderation backlog and actions,
- payment outcomes and failure reasons,
- ownership/authorization denials.

---

## 9. Non-Functional Architecture Controls

### 9.1 Security

- Server-side membership verification gates all protected operations.
- Ownership checks enforced in workflow steps.
- Secret values (RPC endpoints, verification keys) remain server-side.
- Deny-by-default for non-member and cross-seller access.

### 9.2 Reliability

- Idempotent payment verification workflow.
- Retry-safe publish/moderation transitions.
- Deterministic moderation transition responses.

### 9.3 Performance

- Paginated listing/order queries.
- Indexed filters on `seller_member_profile_id`, `status`, `created_at`, `order_id`, `tx_hash`.
- Membership verification cache with strict TTL + invalidation strategy.

---

## 10. Deployment and Environment Strategy

### 10.1 Environments

- `local` -> `staging` -> `production` with same migration lineage.

### 10.2 Required Configuration

- DAO address (governance scope). that the adresses of people in charge of the platform.
- NFT collection identifier.
- ELURC token address + network settings.
- RPC endpoints and confirmation depth policy.

### 10.3 Release Gate for MVP

Must pass E2E:

1. membership verification ->
2. listing publish ->
3. ELURC payment via Phantom ->
4. payment verification ->
5. order completion.

Also must pass automated suites:

- members-only access,
- seller ownership isolation,
- DAO issue intervention,
- ELURC-only enforcement.

---

## 11. Implementation Roadmap (Architecture-First)

1. Create modules: `membership`, `marketplace`, `paymentMeta`, `audit`.
2. Register modules in Medusa config.
3. Generate and run migrations.
4. Implement core workflows (membership, listing, moderation, payment verification).
5. Implement store/admin routes + middleware validation/auth.
6. Integrate custom ELURC payment provider and status endpoint.
7. Add structured logging + audit events.
8. Execute E2E and authorization regression suites.

---

## 12. Open Decisions to Finalize

1. Membership verification cache policy (TTL, refresh triggers, failure fallback).
2. Payment confirmation policy (required confirmations, timeout threshold).
3. MVP tenancy depth:
   - Option A: member-as-seller scope only (strict PRD baseline),
   - Option B: introduce organization boundary metadata now while keeping direct publish.
4. Post-MVP governance pipeline activation (proposal pre-approval mode toggle).

---

## 13. Final Architecture Statement

Stalabard MVP will be implemented as a Medusa v2 modular marketplace backend where membership-gated access, ownership-scoped seller operations, reactive DAO moderation, and ELURC-only non-custodial checkout are enforced server-side through workflow-centric business logic. The architecture prioritizes security boundaries, deterministic state transitions, and auditable governance/payment trails while remaining extensible for future multi-tenant and proposal-first governance evolution.
