# Story 1.2: Enforce Members-Only Access on Protected Routes

Status: ready-for-dev

## Story

As a platform operator,
I want protected routes to reject non-members,
So that marketplace actions are limited to verified community members.

## Acceptance Criteria

**Given** a non-verified or failed-verification user
**When** they call a protected member route
**Then** the API returns forbidden with a deterministic error code
**And** no mutation or protected read is performed.

## Requirements Coverage

**Functional Requirements:** FR-001, FR-005, NFR-001

- **FR-001**: System SHALL allow platform access only to verified community members.
- **FR-005**: System SHALL deny non-member access to protected routes and actions.
- **NFR-001**: Enforce role-based authorization on all custom APIs.

## Tasks / Subtasks

- [ ] Create authentication middleware for member routes (AC: #1)
  - [ ] Implement `authenticateMember` middleware in `src/api/middlewares/authenticate-member.ts`
  - [ ] Check session/bearer token validity
  - [ ] Load member identity from membership module
  - [ ] Verify membership_status is 'verified' or 'active'
  - [ ] Attach member context to request object
  - [ ] Return 401 Unauthorized if no valid session
  - [ ] Return 403 Forbidden if member not verified

- [ ] Create authorization middleware for protected routes (AC: #1)
  - [ ] Implement `requireMembership` middleware
  - [ ] Check member verification status from request context
  - [ ] Return deterministic error codes for different failure modes
  - [ ] Log authorization denial events with correlation_id

- [ ] Apply middleware to all protected store routes (AC: #1)
  - [ ] Apply to `/store/member/*` routes
  - [ ] Apply to `/store/listings` (marketplace browse)
  - [ ] Apply to `/store/cart` and checkout routes
  - [ ] Apply to `/store/orders` routes
  - [ ] Document which routes require membership vs public access

- [ ] Implement deterministic error response structure (AC: #1)
  - [ ] Define error codes: `MEMBER_NOT_VERIFIED`, `MEMBER_VERIFICATION_FAILED`, `MEMBER_NOT_FOUND`
  - [ ] Create error response schema with code, message, details
  - [ ] Ensure consistent error format across all protected routes
  - [ ] Include helpful error messages for frontend handling

- [ ] Add membership status check helper service (AC: #1)
  - [ ] Create `checkMembershipStatus` service method
  - [ ] Query member_identity by user/session context
  - [ ] Return verification status with timestamp
  - [ ] Handle edge cases (expired verification, suspended members)

- [ ] Implement route-level access control configuration (AC: #1)
  - [ ] Define route access matrix (public, member-only, moderator-only)
  - [ ] Create configuration file for route protection rules
  - [ ] Apply middleware conditionally based on route config
  - [ ] Support future role expansion (moderator, admin)

- [ ] Add structured logging for access denials (AC: #1)
  - [ ] Log all authorization failures with member_id (if available)
  - [ ] Log route attempted, timestamp, failure reason
  - [ ] Include correlation_id for request tracing
  - [ ] Emit security event for monitoring

- [ ] Write unit tests for authentication middleware
  - [ ] Test valid member session passes through
  - [ ] Test non-verified member is rejected
  - [ ] Test missing session is rejected
  - [ ] Test expired session handling
  - [ ] Test error response format

- [ ] Write integration tests for protected routes
  - [ ] Test member-only route access with verified member
  - [ ] Test member-only route rejection for non-member
  - [ ] Test member-only route rejection for unverified member
  - [ ] Test public routes remain accessible
  - [ ] Test error codes are deterministic

- [ ] Write security regression tests (AC: #1)
  - [ ] Test cannot bypass middleware with direct service calls
  - [ ] Test cannot access protected routes without verification
  - [ ] Test session tampering is detected
  - [ ] Test rate limiting on failed auth attempts (if implemented)

## Dev Notes

### Architecture Patterns and Constraints

**Middleware Architecture (CRITICAL):**
- Middleware executes BEFORE route handlers
- Chain middleware in correct order: session → authenticate → authorize → validate → handler
- Use `AuthenticatedMedusaRequest<T>` type for protected routes
- Never trust frontend for authorization decisions

**Middleware Chain Pattern:**
```typescript
// Example protected route
export const POST = [
  authenticate("member"), // Session validation
  requireMembership(), // Membership verification check
  validateAndTransformBody(CreateListingSchema),
  // Route handler
]
```

**Route Protection Levels:**
1. **Public routes**: No authentication required (e.g., landing page, docs)
2. **Member routes**: Require verified membership (e.g., `/store/member/*`, `/store/listings`)
3. **Moderator routes**: Require moderator role (e.g., `/admin/moderation/*`)

### Technical Requirements

**Authentication Flow:**
1. Extract session/bearer token from request headers
2. Validate token signature and expiration
3. Load user/customer from Medusa auth context
4. Query membership module for member_identity
5. Verify membership_status === 'verified'
6. Attach member context to request: `req.member`

**Authorization Enforcement:**
- Check membership status on EVERY protected route
- Deny by default (whitelist approach, not blacklist)
- Return 403 Forbidden for verified session but unverified member
- Return 401 Unauthorized for invalid/missing session
- Log all denials for security monitoring

**Error Response Format:**
```typescript
{
  code: "MEMBER_NOT_VERIFIED",
  message: "Access denied: Membership verification required",
  details: {
    wallet_address: "...",
    verification_status: "pending",
    help_url: "/verify-membership"
  }
}
```

**Session Management:**
- Use Medusa's built-in session handling
- Extend session context with member_id after verification
- Session should include: user_id, member_id, verification_status
- Consider session TTL and refresh strategy

### Library and Framework Requirements

**Required Dependencies:**
- `@medusajs/medusa` (v2) - Core framework with auth
- `express` - Middleware support (included with Medusa)
- `zod` - Error schema validation

**Medusa Auth Patterns:**
- Use `authenticate()` middleware from Medusa
- Extend with custom `requireMembership()` middleware
- Use `AuthenticatedMedusaRequest` type for type safety

### File Structure Requirements

**Middleware Files:**
```
src/api/middlewares/
  ├── authenticate-member.ts
  ├── require-membership.ts
  └── index.ts
```

**Route Protection Example:**
```
src/api/store/member/listings/route.ts
src/api/store/listings/route.ts (member-only browse)
src/api/admin/moderation/route.ts (moderator-only)
```

**Configuration:**
```
src/config/route-access.ts (route protection rules)
```

### Testing Requirements

**Unit Test Coverage:**
- Middleware functions in isolation
- Error response formatting
- Member status check logic
- Session validation logic

**Integration Test Coverage:**
- End-to-end protected route access
- Non-member rejection scenarios
- Verified member access scenarios
- Error code consistency across routes

**Security Test Scenarios:**
- Attempt to access `/store/member/listings` without session → 401
- Attempt to access `/store/member/listings` with unverified member → 403
- Attempt to access `/store/listings` without membership → 403
- Verified member accesses `/store/member/listings` → 200
- Tampered session token → 401
- Expired session → 401

**Test Frameworks:**
- Jest for unit tests
- Supertest for API integration tests
- Test fixtures for verified/unverified members

### Observability Requirements

**Structured Logging Fields:**
- `route_path` - Protected route attempted
- `member_id` - Member attempting access (if available)
- `wallet_address` - Wallet address (if available)
- `verification_status` - Current member status
- `denial_reason` - Why access was denied
- `correlation_id` - Request tracing
- `timestamp` - ISO 8601 format

**Security Events:**
- Event: `access.denied.no_session`
  - Payload: route_path, ip_address, timestamp
- Event: `access.denied.unverified_member`
  - Payload: route_path, member_id, verification_status, timestamp
- Event: `access.granted.verified_member`
  - Payload: route_path, member_id, timestamp

**Operational Metrics:**
- Access denial rate by route
- Access denial rate by reason code
- Member verification success rate
- Average time to verification

### Project Structure Notes

**Alignment with Unified Project Structure:**
- Place middleware in `src/api/middlewares/`
- Follow Medusa v2 middleware conventions
- Use TypeScript for type safety
- Export middleware from index for easy imports

**Middleware Naming Convention:**
- Use camelCase: `authenticateMember`, `requireMembership`
- Prefix with action verb: `authenticate`, `require`, `validate`
- Keep names descriptive and consistent

### Dependencies on Other Stories

**Depends on Story 1.1:**
- Requires `membership` module to be implemented
- Requires `member_identity` model with `membership_status` field
- Requires ability to query member verification status

**Enables Future Stories:**
- Story 1.3: Dual buyer/seller capability (uses same middleware)
- Story 1.4: Seller ownership scope (builds on member context)
- All Epic 2+ stories (all require member authentication)

### References

**Source Documents:**
- [PRD: Section 5.1 - Membership and Access Control](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#51-membership-and-access-control)
- [Architecture: Section 4.2 - Authorization Model](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#42-authorization-model)
- [Architecture: Section 4.3 - Enforcement Location](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#43-enforcement-location)
- [Architecture: Section 6.2 - API Standards](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#62-api-standards)
- [Architecture: Section 9.1 - Security](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#91-security)
- [Epic 1: Story 1.2](c:/Projects/Stalabard/_bmad-output/planning-artifacts/epics.md#story-12-enforce-members-only-access-on-protected-routes)

**External Documentation:**
- Medusa v2 Authentication: https://docs.medusajs.com/development/api-routes/authentication
- Medusa v2 Middleware: https://docs.medusajs.com/development/api-routes/middlewares

## Dev Agent Record

### Agent Model Used

_To be filled by dev agent_

### Debug Log References

_To be filled by dev agent during implementation_

### Completion Notes List

_To be filled by dev agent upon completion_

### File List

_To be filled by dev agent with all created/modified files_
