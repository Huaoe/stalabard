# Story 1.4: Enforce Seller Ownership Scope Guard

Status: ready-for-dev

## Story

As a seller,
I want access to be restricted to my own seller resources,
So that no seller can read or mutate another seller's listings or seller operations.

## Acceptance Criteria

**Given** seller A requests seller B resources
**When** the ownership guard evaluates seller_member_profile_id
**Then** access is denied with forbidden response
**And** an authorization denial event is logged.

## Requirements Coverage

**Functional Requirements:** FR-004, NFR-002

- **FR-004**: Seller views SHALL only expose that seller's own listings and seller operations.
- **NFR-002**: Enforce server-side ownership checks for seller-scoped resources.

## Tasks / Subtasks

- [ ] Create ownership guard middleware (AC: #1)
  - [ ] Implement `requireOwnership` middleware in `src/api/middlewares/require-ownership.ts`
  - [ ] Extract resource owner from request (seller_member_profile_id)
  - [ ] Compare with authenticated member's member_profile_id
  - [ ] Return 403 Forbidden if ownership check fails
  - [ ] Attach ownership context to request for downstream use

- [ ] Implement listing ownership verification (AC: #1)
  - [ ] Create `verifyListingOwnership(listing_id, member_profile_id)` helper
  - [ ] Query listing by ID and check seller_member_profile_id
  - [ ] Return boolean ownership result
  - [ ] Handle non-existent listing (404 vs 403 decision)
  - [ ] Cache ownership checks for performance (optional)

- [ ] Apply ownership guard to seller listing routes (AC: #1)
  - [ ] Apply to `POST /store/member/listings/:id` (update listing)
  - [ ] Apply to `POST /store/member/listings/:id/publish` (publish listing)
  - [ ] Apply to `DELETE /store/member/listings/:id` (delete listing)
  - [ ] Apply to `GET /store/member/listings/:id` (view own listing detail)
  - [ ] Ensure middleware runs AFTER authentication, BEFORE handler

- [ ] Implement seller-scoped query filters (AC: #1)
  - [ ] Add `filterBySeller(member_profile_id)` query helper
  - [ ] Apply to `GET /store/member/listings` (list own listings)
  - [ ] Apply to `GET /store/member/orders` (orders for own listings)
  - [ ] Ensure queries ALWAYS include seller filter
  - [ ] Prevent query parameter injection to bypass filter

- [ ] Create ownership denial error responses (AC: #1)
  - [ ] Define error code: `OWNERSHIP_DENIED`
  - [ ] Include resource type and ID in error details
  - [ ] Provide helpful message without leaking resource existence
  - [ ] Return 403 Forbidden (not 404) for ownership failures

- [ ] Add authorization denial logging (AC: #1)
  - [ ] Log all ownership check failures
  - [ ] Include: member_id, resource_type, resource_id, attempted_action
  - [ ] Include correlation_id for request tracing
  - [ ] Emit security event for monitoring

- [ ] Implement workflow-level ownership checks (AC: #1)
  - [ ] Add ownership validation step to update-listing workflow
  - [ ] Add ownership validation step to publish-listing workflow
  - [ ] Add ownership validation step to delete-listing workflow
  - [ ] Ensure workflows fail fast on ownership violation
  - [ ] Include ownership context in workflow execution logs

- [ ] Create seller resource access helper service (AC: #1)
  - [ ] Implement `getSellerResource(resource_type, resource_id, member_profile_id)`
  - [ ] Centralize ownership check logic
  - [ ] Return resource if owned, throw error if not
  - [ ] Support multiple resource types (listings, seller orders)

- [ ] Write unit tests for ownership guard
  - [ ] Test owner can access own resource
  - [ ] Test non-owner is denied access to resource
  - [ ] Test error response format
  - [ ] Test ownership check with non-existent resource
  - [ ] Test ownership check with invalid member context

- [ ] Write integration tests for seller-scoped routes
  - [ ] Test seller A can update own listing
  - [ ] Test seller A cannot update seller B's listing
  - [ ] Test seller A can only see own listings in list view
  - [ ] Test seller A cannot access seller B's listing detail
  - [ ] Test ownership denial is logged

- [ ] Write security regression tests (AC: #1)
  - [ ] Test cannot bypass ownership check via query parameters
  - [ ] Test cannot bypass ownership check via workflow direct calls
  - [ ] Test cannot access resources by guessing IDs
  - [ ] Test ownership checks are enforced at both route and workflow levels

## Dev Notes

### Architecture Patterns and Constraints

**Ownership Guard Pattern (CRITICAL):**
- Ownership checks MUST be enforced server-side (never trust frontend)
- Check ownership at BOTH route middleware AND workflow steps
- Deny by default: if ownership cannot be verified, deny access
- Use 403 Forbidden (not 404) to avoid leaking resource existence

**Defense in Depth:**
```
Request → Auth Middleware → Ownership Guard → Route Handler → Workflow
                                                                   ↓
                                                          Ownership Check Step
```

**Ownership Check Locations:**
1. **Route Middleware**: Fast fail before handler execution
2. **Workflow Steps**: Validate ownership before mutations
3. **Query Filters**: Automatically scope all seller queries

**Resource Ownership Model:**
- Listings: `seller_member_profile_id` field
- Seller Orders: Derived from listing ownership
- Future seller resources: Follow same pattern

### Technical Requirements

**Ownership Verification Logic:**
```typescript
async function verifyOwnership(
  resourceType: "listing" | "seller_order",
  resourceId: string,
  memberProfileId: string
): Promise<boolean> {
  // Query resource and check owner field
  const resource = await getResource(resourceType, resourceId);
  if (!resource) return false;
  
  return resource.seller_member_profile_id === memberProfileId;
}
```

**Middleware Implementation:**
```typescript
export function requireOwnership(resourceType: string) {
  return async (req, res, next) => {
    const resourceId = req.params.id;
    const memberProfileId = req.member.member_profile_id;
    
    const isOwner = await verifyOwnership(resourceType, resourceId, memberProfileId);
    
    if (!isOwner) {
      // Log denial
      logger.warn("Ownership denied", {
        member_id: req.member.member_id,
        resource_type: resourceType,
        resource_id: resourceId,
        correlation_id: req.correlation_id
      });
      
      return res.status(403).json({
        code: "OWNERSHIP_DENIED",
        message: "Access denied: You do not own this resource"
      });
    }
    
    next();
  };
}
```

**Query Filter Pattern:**
```typescript
// ALWAYS apply seller filter to queries
async function listOwnListings(memberProfileId: string, filters: any) {
  return await listingService.list({
    ...filters,
    seller_member_profile_id: memberProfileId // REQUIRED
  });
}
```

**Workflow Ownership Step:**
```typescript
// In update-listing workflow
const validateOwnershipStep = createStep(
  "validate-listing-ownership",
  async ({ listing_id, member_profile_id }) => {
    const listing = await listingService.retrieve(listing_id);
    
    if (listing.seller_member_profile_id !== member_profile_id) {
      throw new MedusaError(
        MedusaError.Types.NOT_ALLOWED,
        "Ownership verification failed"
      );
    }
    
    return { verified: true };
  }
);
```

### Library and Framework Requirements

**Required Dependencies:**
- `@medusajs/medusa` (v2) - Core framework
- `@medusajs/utils` - MedusaError types
- Membership module (from Story 1.1)
- Marketplace module (from Epic 2)

**No Additional Authorization Libraries:**
- Use built-in Medusa patterns
- Implement custom ownership logic
- Avoid over-engineering with RBAC frameworks

### File Structure Requirements

**Ownership Middleware:**
```
src/api/middlewares/
  ├── require-ownership.ts
  └── index.ts
```

**Ownership Helper Service:**
```
src/utils/
  ├── ownership.ts
  └── index.ts
```

**Workflow Ownership Steps:**
```
src/workflows/marketplace/steps/
  ├── validate-listing-ownership.ts
  └── index.ts
```

### Testing Requirements

**Unit Test Coverage:**
- Ownership verification logic
- Middleware ownership checks
- Query filter application
- Error response formatting

**Integration Test Coverage:**
- Seller A updates own listing (success)
- Seller A attempts to update Seller B's listing (403)
- Seller A lists own listings (only sees own)
- Seller A attempts to access Seller B's listing detail (403)
- Ownership denial is logged correctly

**Security Test Scenarios:**
1. **Cross-Seller Access Attempt:**
   - Seller A creates listing (listing_id: 123)
   - Seller B attempts `POST /store/member/listings/123` → 403
   - Seller B attempts `GET /store/member/listings/123` → 403
   - Seller B attempts workflow call to update listing 123 → Error

2. **Query Injection Attempt:**
   - Seller A attempts `GET /store/member/listings?seller_member_profile_id=seller_b_id`
   - Query parameter is ignored
   - Only Seller A's listings are returned

3. **Resource Enumeration:**
   - Seller A attempts to access listing IDs 1-1000
   - All non-owned listings return 403
   - No information leakage about resource existence

**Test Frameworks:**
- Jest for unit tests
- Supertest for API integration tests
- Test fixtures for multiple sellers with listings

### Observability Requirements

**Structured Logging Fields:**
- `member_id` - Member attempting access
- `resource_type` - Type of resource (listing, seller_order)
- `resource_id` - ID of resource
- `resource_owner_id` - Actual owner member_profile_id
- `attempted_action` - Action attempted (read, update, delete)
- `denial_reason` - Why ownership check failed
- `correlation_id` - Request tracing

**Security Events:**
- Event: `ownership.denied`
  - Payload: member_id, resource_type, resource_id, attempted_action, timestamp
- Event: `ownership.verified`
  - Payload: member_id, resource_type, resource_id, action, timestamp

**Operational Metrics:**
- Ownership denial rate by resource type
- Ownership denial rate by member
- Most frequently attempted unauthorized resources
- Average ownership check duration

### Project Structure Notes

**Alignment with Unified Project Structure:**
- Place ownership logic in centralized utilities
- Apply middleware consistently across seller routes
- Enforce ownership at both API and workflow layers
- Follow Medusa v2 authorization patterns

**Centralized Ownership Logic:**
- ✅ Create reusable ownership verification functions
- ✅ Apply consistently across all seller resources
- ✅ Enforce at multiple layers (middleware + workflow)
- ❌ Don't duplicate ownership logic in each route
- ❌ Don't rely on frontend for ownership filtering

### Dependencies on Other Stories

**Depends on:**
- Story 1.1: Member verification and member_profile model
- Story 1.2: Member authentication middleware
- Story 1.3: Dual buyer/seller capability (member context)
- Epic 2 Stories: Listing entity with seller_member_profile_id field

**Enables:**
- Epic 2: Secure seller listing management
- Epic 3: Prevents cross-seller order access
- Story 1.5: Ownership regression tests

### References

**Source Documents:**
- [PRD: Section 1.1 - Objectives (Sellers only access own resources)](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#11-objectives)
- [PRD: Section 2.1 - Community Member (Manages only own seller resources)](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#21-community-member-seller--buyer)
- [PRD: FR-004, FR-021](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#51-membership-and-access-control)
- [PRD: NFR-002](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#61-security)
- [PRD: Journey C - Seller Ownership Scope](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#journey-c-seller-ownership-scope)
- [Architecture: Section 4.2 - Authorization Model (Seller scope rule)](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#42-authorization-model)
- [Architecture: Section 4.3 - Enforcement Location (Workflow steps enforce ownership)](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#43-enforcement-location)
- [Architecture: Section 9.1 - Security (Ownership checks enforced in workflow steps)](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#91-security)
- [Epic 1: Story 1.4](c:/Projects/Stalabard/_bmad-output/planning-artifacts/epics.md#story-14-enforce-seller-ownership-scope-guard)

**External Documentation:**
- Medusa v2 Error Handling: https://docs.medusajs.com/development/fundamentals/error-handling
- Medusa v2 Workflows: https://docs.medusajs.com/development/workflows/create

## Dev Agent Record

### Agent Model Used

_To be filled by dev agent_

### Debug Log References

_To be filled by dev agent during implementation_

### Completion Notes List

_To be filled by dev agent upon completion_

### File List

_To be filled by dev agent with all created/modified files_
