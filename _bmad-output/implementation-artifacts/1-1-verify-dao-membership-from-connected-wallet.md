# Story 1.1: Verify DAO Membership from Connected Wallet

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a community member,
I want the platform to verify my DAO NFT badge ownership server-side when I connect my wallet,
So that only verified members can access protected marketplace features.

## Acceptance Criteria

**Given** a user connects a wallet address
**When** the membership verification workflow executes
**Then** DAO NFT ownership is verified server-side against configured governance values
**And** member identity status is persisted with verification timestamp.

## Requirements Coverage

**Functional Requirements:** FR-001, FR-002, NFR-005

- **FR-001**: System SHALL allow platform access only to verified community members.
- **FR-002**: System SHALL verify membership using DAO NFT badge ownership.
- **NFR-005**: Membership verification checks MUST be server-side enforced for protected routes.

## Tasks / Subtasks

- [ ] Create `membership` module with data models (AC: #1)
  - [ ] Define `member_identity` model with wallet_address, membership_status, verified_at, source fields
  - [ ] Define `member_profile` model with member_identity_id, display_name, avatar_url, metadata fields
  - [ ] Create module service with CRUD operations
  - [ ] Register module in Medusa configuration
  - [ ] Generate and apply database migration

- [ ] Implement `verify-membership-workflow` (AC: #1)
  - [ ] Create workflow definition in `src/workflows/membership/verify-membership.ts`
  - [ ] Add step to validate wallet address format
  - [ ] Add step to query blockchain RPC for NFT ownership
  - [ ] Add step to verify against configured DAO collection: `3e22667e998143beef529eda8c84ee394a838aa705716c3b6fe0b3d5f913ac4c`
  - [ ] Add step to create/update member_identity with verification result
  - [ ] Add step to create member_profile if new member
  - [ ] Add compensation steps for rollback scenarios
  - [ ] Ensure idempotent behavior for retry safety

- [ ] Create membership verification API route (AC: #1)
  - [ ] Implement `POST /store/member/verify` endpoint
  - [ ] Add Zod schema validation for wallet address input
  - [ ] Use `AuthenticatedMedusaRequest<T>` pattern
  - [ ] Invoke verify-membership-workflow
  - [ ] Return deterministic verification status response
  - [ ] Handle error cases with explicit error codes

- [ ] Add blockchain RPC integration layer (AC: #1)
  - [ ] Create RPC client service for NFT ownership queries
  - [ ] Implement NFT ownership check against DAO collection
  - [ ] Add retry logic for RPC failures
  - [ ] Add timeout handling
  - [ ] Cache verification results with TTL (consider Redis)

- [ ] Implement server-side configuration (AC: #1)
  - [ ] Add DAO address to environment config: `D6d8TZrNFwg3QG97JLLfTjgdJequ84wMhQ8f12UW56Rq`
  - [ ] Add NFT collection identifier to config: `3e22667e998143beef529eda8c84ee394a838aa705716c3b6fe0b3d5f913ac4c`
  - [ ] Add RPC endpoint configuration
  - [ ] Ensure secrets remain server-side only (NFR-003)

- [ ] Add structured logging and audit events (AC: #1)
  - [ ] Log membership verification attempts with wallet_address, timestamp
  - [ ] Log verification success/failure with member_id correlation
  - [ ] Emit structured event for successful verification
  - [ ] Include correlation_id for request tracing (NFR-030)

- [ ] Write unit tests for membership module
  - [ ] Test member_identity creation and updates
  - [ ] Test member_profile linkage
  - [ ] Test service CRUD operations

- [ ] Write integration tests for verify-membership-workflow
  - [ ] Test successful NFT ownership verification
  - [ ] Test failed verification (no NFT ownership)
  - [ ] Test invalid wallet address handling
  - [ ] Test RPC failure scenarios
  - [ ] Test idempotent retry behavior

- [ ] Write API route tests
  - [ ] Test valid wallet verification request
  - [ ] Test invalid payload rejection
  - [ ] Test deterministic error responses

## Dev Notes

### Architecture Patterns and Constraints

**Medusa v2 Architecture Pattern (CRITICAL):**
- Follow strict module -> workflow -> API route pattern
- ALL business logic MUST be in workflows, NOT in route handlers
- Routes are thin HTTP interfaces only
- Use `AuthenticatedMedusaRequest<T>` for protected routes
- Validate request bodies with Zod + `validateAndTransformBody` middleware

**Module Structure:**
```
src/modules/membership/
  ├── models/
  │   ├── member-identity.ts
  │   └── member-profile.ts
  ├── service.ts
  └── index.ts
```

**Workflow Structure:**
```
src/workflows/membership/
  ├── verify-membership.ts
  └── steps/
      ├── validate-wallet.ts
      ├── check-nft-ownership.ts
      ├── update-member-identity.ts
      └── create-member-profile.ts
```

**API Route Structure:**
```
src/api/store/member/verify/route.ts
```

### Technical Requirements

**Database Schema (PostgreSQL):**
- Use Medusa module data model patterns
- Define proper indexes on `wallet_address` (unique), `membership_status`, `verified_at`
- Use module-link pattern for cross-module relationships

**Blockchain Integration:**
- Use Solana Web3.js or equivalent for RPC calls
- Query NFT ownership for specific collection ID
- Handle network failures gracefully with retries
- Consider rate limiting for RPC endpoints

**Caching Strategy (Redis recommended):**
- Cache successful verification results with TTL
- Invalidate cache on membership status changes
- Never bypass server-side verification for protected routes
- Cache key pattern: `member:verification:{wallet_address}`

**Security Enforcement:**
- ALL membership checks MUST be server-side (NFR-005)
- Never trust frontend for security boundaries
- Deny-by-default for non-verified members
- No sensitive credentials in public runtime variables (NFR-003)

### Library and Framework Requirements

**Required Dependencies:**
- `@medusajs/medusa` (v2) - Core framework
- `@medusajs/workflows-sdk` - Workflow orchestration
- `zod` - Request validation
- `@solana/web3.js` or equivalent - Blockchain RPC integration
- `ioredis` (recommended) - Redis client for caching

**Configuration Values:**
- DAO Address: `D6d8TZrNFwg3QG97JLLfTjgdJequ84wMhQ8f12UW56Rq`
- NFT Collection: `3e22667e998143beef529eda8c84ee394a838aa705716c3b6fe0b3d5f913ac4c`
- RPC Endpoint: (to be configured per environment)

### File Structure Requirements

**Module Registration:**
- Register `membership` module in `medusa-config.js`
- Ensure module is loaded before dependent modules

**Migration Files:**
- Generate migration with timestamp prefix
- Include both `member_identity` and `member_profile` tables
- Add indexes for performance (wallet_address, membership_status)
- Follow Medusa migration lineage pattern: local -> staging -> production

**Workflow Registration:**
- Export workflow from `src/workflows/membership/index.ts`
- Register in workflow container
- Ensure workflow is discoverable by API routes

### Testing Requirements

**Unit Test Coverage:**
- Module service methods (create, update, find)
- Workflow step functions in isolation
- RPC client methods with mocked responses

**Integration Test Coverage:**
- End-to-end workflow execution
- Database persistence verification
- RPC integration with test fixtures
- Error handling and compensation flows

**Test Frameworks:**
- Use Medusa testing utilities
- Jest for unit and integration tests
- Supertest for API route testing

**Test Data:**
- Valid wallet addresses with NFT ownership
- Valid wallet addresses without NFT ownership
- Invalid wallet address formats
- RPC timeout/failure scenarios

### Observability Requirements

**Structured Logging Fields (NFR-030):**
- `wallet_address` - User wallet being verified
- `member_id` - Member identity ID (after creation)
- `verification_status` - success/failure/pending
- `correlation_id` - Request tracing identifier
- `timestamp` - ISO 8601 format
- `duration_ms` - Verification duration

**Audit Events (NFR-031):**
- Event: `membership.verified`
  - Payload: wallet_address, member_id, verified_at
- Event: `membership.verification_failed`
  - Payload: wallet_address, failure_reason, attempted_at

**Operational Metrics:**
- Verification success rate
- Average verification duration
- RPC failure rate
- Cache hit rate

### Project Structure Notes

**Alignment with Unified Project Structure:**
- Follow Medusa v2 project structure conventions
- Place custom modules in `src/modules/`
- Place workflows in `src/workflows/`
- Place API routes in `src/api/store/` or `src/api/admin/`
- Use camelCase for module names: `membership` (not `member-ship` or `Membership`)

**Module Naming Convention:**
- Use camelCase only: `membership`, `marketplace`, `paymentMeta`, `audit`
- Avoid kebab-case or PascalCase for module folder names
- Follow Medusa module naming standards

### References

**Source Documents:**
- [PRD: Section 5.1 - Membership and Access Control](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#51-membership-and-access-control)
- [PRD: Section 1.2 - Constraints (DAO Address and NFT Collection)](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#12-constraints)
- [Architecture: Section 3.1.A - Membership Module](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#a-membership-module)
- [Architecture: Section 4 - Access Control Architecture](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#4-access-control-architecture)
- [Architecture: Section 5 - Workflow Architecture (verify-membership-workflow)](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#5-workflow-architecture-mutation-flows)
- [Architecture: Section 9.1 - Security (Server-side verification)](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#91-security)
- [Epic 1: Story 1.1](c:/Projects/Stalabard/_bmad-output/planning-artifacts/epics.md#story-11-verify-dao-membership-from-connected-wallet)

**External Documentation:**
- Medusa v2 Module Development: https://docs.medusajs.com/development/modules/create
- Medusa v2 Workflows: https://docs.medusajs.com/development/workflows/create
- Medusa v2 API Routes: https://docs.medusajs.com/development/api-routes/create
- Solana Web3.js: https://solana-labs.github.io/solana-web3.js/

## Dev Agent Record

### Agent Model Used

_To be filled by dev agent_

### Debug Log References

_To be filled by dev agent during implementation_

### Completion Notes List

_To be filled by dev agent upon completion_

### File List

_To be filled by dev agent with all created/modified files_
