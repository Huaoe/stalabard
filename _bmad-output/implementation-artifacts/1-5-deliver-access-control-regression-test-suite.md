# Story 1.5: Deliver Access-Control Regression Test Suite

Status: ready-for-dev

## Story

As a development team,
I want regression tests for membership gating and ownership boundaries,
So that security constraints remain enforced across changes.

## Acceptance Criteria

**Given** the test suite is executed
**When** scenarios include non-member access and cross-seller requests
**Then** all unauthorized cases fail with expected status codes
**And** authorized verified-member cases pass consistently.

## Requirements Coverage

**Functional Requirements:** FR-001, FR-004, FR-005, NFR-001, NFR-002

- **FR-001**: System SHALL allow platform access only to verified community members.
- **FR-004**: Seller views SHALL only expose that seller's own listings and seller operations.
- **FR-005**: System SHALL deny non-member access to protected routes and actions.
- **NFR-001**: Enforce role-based authorization on all custom APIs.
- **NFR-002**: Enforce server-side ownership checks for seller-scoped resources.

## Tasks / Subtasks

- [ ] Set up test infrastructure (AC: #1)
  - [ ] Create test database configuration
  - [ ] Set up test fixtures for members, listings, orders
  - [ ] Create test helper utilities for auth and requests
  - [ ] Configure test environment variables
  - [ ] Set up test data cleanup between test runs

- [ ] Create member verification test fixtures (AC: #1)
  - [ ] Fixture: Verified member A with wallet and NFT ownership
  - [ ] Fixture: Verified member B with wallet and NFT ownership
  - [ ] Fixture: Unverified member (no NFT ownership)
  - [ ] Fixture: Non-member (no verification attempt)
  - [ ] Fixture: Member with expired verification
  - [ ] Helper: Generate test wallet addresses
  - [ ] Helper: Mock NFT ownership RPC responses

- [ ] Implement membership gating regression tests (AC: #1)
  - [ ] Test: Non-member cannot access `/store/listings` → 403
  - [ ] Test: Non-member cannot access `/store/member/listings` → 401/403
  - [ ] Test: Unverified member cannot access protected routes → 403
  - [ ] Test: Verified member can access `/store/listings` → 200
  - [ ] Test: Verified member can access `/store/member/listings` → 200
  - [ ] Test: Session without membership verification is rejected → 401
  - [ ] Test: Expired membership verification is rejected → 403

- [ ] Implement ownership boundary regression tests (AC: #1)
  - [ ] Test: Seller A can view own listing → 200
  - [ ] Test: Seller A cannot view Seller B's listing → 403
  - [ ] Test: Seller A can update own listing → 200
  - [ ] Test: Seller A cannot update Seller B's listing → 403
  - [ ] Test: Seller A can publish own listing → 200
  - [ ] Test: Seller A cannot publish Seller B's listing → 403
  - [ ] Test: Seller A can delete own listing → 200
  - [ ] Test: Seller A cannot delete Seller B's listing → 403

- [ ] Implement seller-scoped query regression tests (AC: #1)
  - [ ] Test: `GET /store/member/listings` returns only own listings
  - [ ] Test: Seller A cannot see Seller B's listings in list view
  - [ ] Test: Query parameters cannot bypass seller filter
  - [ ] Test: `GET /store/member/orders` returns only orders for own listings
  - [ ] Test: Seller cannot see orders for other sellers' listings

- [ ] Implement cross-seller access attempt tests (AC: #1)
  - [ ] Test: Seller A creates listing, Seller B attempts access → 403
  - [ ] Test: Seller A publishes listing, Seller B attempts unpublish → 403
  - [ ] Test: Seller A receives order, Seller B attempts to view → 403
  - [ ] Test: All cross-seller attempts are logged as security events

- [ ] Implement workflow-level authorization tests (AC: #1)
  - [ ] Test: Update-listing workflow rejects non-owner
  - [ ] Test: Publish-listing workflow rejects non-owner
  - [ ] Test: Delete-listing workflow rejects non-owner
  - [ ] Test: Workflows fail fast on ownership violation
  - [ ] Test: Workflow errors include ownership denial reason

- [ ] Implement error response consistency tests (AC: #1)
  - [ ] Test: All membership denials return consistent error format
  - [ ] Test: All ownership denials return consistent error format
  - [ ] Test: Error codes are deterministic (same input → same error code)
  - [ ] Test: Error messages do not leak sensitive information
  - [ ] Test: 401 vs 403 status codes are used correctly

- [ ] Implement security event logging tests (AC: #1)
  - [ ] Test: Membership denial is logged with correlation_id
  - [ ] Test: Ownership denial is logged with resource details
  - [ ] Test: Successful authorization is logged (optional)
  - [ ] Test: Log entries include all required fields
  - [ ] Test: Logs can be queried by member_id and resource_id

- [ ] Create test utilities and helpers (AC: #1)
  - [ ] Helper: `createVerifiedMember()` - Creates member with verified status
  - [ ] Helper: `createUnverifiedMember()` - Creates member without verification
  - [ ] Helper: `createListingForMember(member)` - Creates listing owned by member
  - [ ] Helper: `authenticateAsMember(member)` - Returns auth headers for member
  - [ ] Helper: `expectOwnershipDenied(response)` - Asserts 403 with correct error
  - [ ] Helper: `expectMembershipRequired(response)` - Asserts 403 with correct error

- [ ] Implement continuous regression test execution (AC: #1)
  - [ ] Add tests to CI/CD pipeline
  - [ ] Configure tests to run on every PR
  - [ ] Set up test coverage reporting
  - [ ] Define minimum coverage thresholds (e.g., 80% for auth/ownership)
  - [ ] Block merges if regression tests fail

- [ ] Document test scenarios and expected outcomes (AC: #1)
  - [ ] Create test scenario matrix (member type × action × expected result)
  - [ ] Document test data setup requirements
  - [ ] Document how to run tests locally
  - [ ] Document how to add new regression tests
  - [ ] Include examples of test failures and debugging

## Dev Notes

### Architecture Patterns and Constraints

**Regression Test Strategy (CRITICAL):**
- Tests MUST cover all security boundaries (membership + ownership)
- Tests MUST run automatically on every code change
- Tests MUST fail if any security constraint is violated
- Tests MUST be deterministic (no flaky tests)

**Test Pyramid for Security:**
```
         E2E Security Flows (5-10 tests)
              ↑
    Integration Tests (30-50 tests)
              ↑
      Unit Tests (100+ tests)
```

**Test Coverage Requirements:**
- Membership gating: 100% of protected routes
- Ownership boundaries: 100% of seller-scoped routes
- Error responses: All error codes and formats
- Security events: All denial scenarios

### Technical Requirements

**Test Database Setup:**
- Use separate test database (not development or production)
- Reset database between test runs for isolation
- Seed test data before each test suite
- Clean up test data after each test suite

**Test Fixtures:**
```typescript
// Member fixtures
const verifiedMemberA = {
  wallet_address: "0xVerifiedA...",
  membership_status: "verified",
  verified_at: new Date(),
};

const verifiedMemberB = {
  wallet_address: "0xVerifiedB...",
  membership_status: "verified",
  verified_at: new Date(),
};

const unverifiedMember = {
  wallet_address: "0xUnverified...",
  membership_status: "pending",
  verified_at: null,
};

// Listing fixtures
const listingOwnedByA = {
  seller_member_profile_id: memberA.member_profile_id,
  title: "Test Listing A",
  status: "published",
};
```

**Test Helpers:**
```typescript
// Authentication helper
async function authenticateAsMember(member: Member) {
  const response = await request(app)
    .post("/auth/session")
    .send({ wallet_address: member.wallet_address });
  
  return {
    Cookie: response.headers["set-cookie"],
    Authorization: `Bearer ${response.body.token}`,
  };
}

// Ownership denial assertion
function expectOwnershipDenied(response: Response) {
  expect(response.status).toBe(403);
  expect(response.body.code).toBe("OWNERSHIP_DENIED");
  expect(response.body.message).toContain("do not own");
}

// Membership required assertion
function expectMembershipRequired(response: Response) {
  expect(response.status).toBe(403);
  expect(response.body.code).toBe("MEMBER_NOT_VERIFIED");
}
```

### Library and Framework Requirements

**Required Dependencies:**
- `jest` - Test framework
- `supertest` - HTTP API testing
- `@medusajs/medusa` (v2) - Core framework
- `pg` - PostgreSQL client for test database
- `faker` or `@faker-js/faker` - Test data generation

**Test Configuration:**
```javascript
// jest.config.js
module.exports = {
  testEnvironment: "node",
  testMatch: ["**/__tests__/security/**/*.test.ts"],
  setupFilesAfterEnv: ["<rootDir>/test/setup.ts"],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};
```

### File Structure Requirements

**Test Directory Structure:**
```
test/
  ├── setup.ts (test environment setup)
  ├── fixtures/
  │   ├── members.ts
  │   ├── listings.ts
  │   └── index.ts
  ├── helpers/
  │   ├── auth.ts
  │   ├── assertions.ts
  │   └── index.ts
  └── security/
      ├── membership-gating.test.ts
      ├── ownership-boundaries.test.ts
      ├── seller-scoped-queries.test.ts
      ├── cross-seller-access.test.ts
      ├── workflow-authorization.test.ts
      └── error-responses.test.ts
```

### Testing Requirements

**Membership Gating Test Scenarios:**
1. Non-member access to protected routes → 401/403
2. Unverified member access to protected routes → 403
3. Verified member access to protected routes → 200
4. Expired verification access attempt → 403
5. Session without membership → 401

**Ownership Boundary Test Scenarios:**
1. Owner access to own resource → 200
2. Non-owner access to resource → 403
3. Owner update to own resource → 200
4. Non-owner update to resource → 403
5. Query returns only owned resources → 200

**Cross-Seller Access Test Scenarios:**
1. Seller A creates listing → Seller B attempts read → 403
2. Seller A creates listing → Seller B attempts update → 403
3. Seller A creates listing → Seller B attempts delete → 403
4. Seller A publishes listing → Seller B attempts unpublish → 403

**Error Response Test Scenarios:**
1. Membership denial error format is consistent
2. Ownership denial error format is consistent
3. Error codes are deterministic
4. Error messages do not leak information
5. HTTP status codes are correct (401 vs 403)

**Test Execution:**
```bash
# Run all security regression tests
npm test -- test/security

# Run specific test suite
npm test -- test/security/membership-gating.test.ts

# Run with coverage
npm test -- --coverage test/security

# Run in watch mode during development
npm test -- --watch test/security
```

### Observability Requirements

**Test Execution Logging:**
- Log test suite start/end times
- Log individual test pass/fail status
- Log test coverage metrics
- Log any test failures with stack traces

**CI/CD Integration:**
- Run tests on every commit
- Run tests on every PR
- Block merge if tests fail
- Report coverage to code review tools

**Test Metrics:**
- Total test count
- Pass/fail rate
- Test execution duration
- Coverage percentage by module

### Project Structure Notes

**Alignment with Unified Project Structure:**
- Place tests in `test/` directory (not `__tests__` inline)
- Organize by security domain (membership, ownership)
- Use shared fixtures and helpers
- Follow Medusa v2 testing conventions

**Test Organization:**
- ✅ Group tests by security concern
- ✅ Use descriptive test names
- ✅ Keep tests isolated and independent
- ✅ Use fixtures for consistent test data
- ❌ Don't share state between tests
- ❌ Don't skip security tests

### Dependencies on Other Stories

**Depends on:**
- Story 1.1: Member verification implementation
- Story 1.2: Member authentication middleware
- Story 1.3: Dual buyer/seller capability
- Story 1.4: Seller ownership scope guard
- Epic 2: Listing entity and routes

**Enables:**
- Continuous security validation
- Safe refactoring with confidence
- Regression prevention for future changes
- Security compliance verification

### References

**Source Documents:**
- [PRD: Section 5.1 - Membership and Access Control](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#51-membership-and-access-control)
- [PRD: Section 6.1 - Security NFRs](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#61-security)
- [PRD: Section 8.3 - Release Readiness Criteria (Test suites)](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#83-release-readiness-criteria)
- [PRD: Journey C - Seller Ownership Scope](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#journey-c-seller-ownership-scope)
- [Architecture: Section 4 - Access Control Architecture](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#4-access-control-architecture)
- [Architecture: Section 9.1 - Security](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#91-security)
- [Architecture: Section 10.3 - Release Gate for MVP](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#103-release-gate-for-mvp)
- [Epic 1: Story 1.5](c:/Projects/Stalabard/_bmad-output/planning-artifacts/epics.md#story-15-deliver-access-control-regression-test-suite)

**External Documentation:**
- Jest Testing Framework: https://jestjs.io/docs/getting-started
- Supertest API Testing: https://github.com/visionmedia/supertest
- Medusa v2 Testing: https://docs.medusajs.com/development/testing

## Dev Agent Record

### Agent Model Used

_To be filled by dev agent_

### Debug Log References

_To be filled by dev agent during implementation_

### Completion Notes List

_To be filled by dev agent upon completion_

### File List

_To be filled by dev agent with all created/modified files_
