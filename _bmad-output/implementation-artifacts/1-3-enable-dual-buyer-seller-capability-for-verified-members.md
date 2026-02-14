# Story 1.3: Enable Dual Buyer/Seller Capability for Verified Members

Status: ready-for-dev

## Story

As a verified member,
I want one identity to access both buyer and seller capabilities,
So that I can participate in both sides of the marketplace without role conflicts.

## Acceptance Criteria

**Given** a verified member profile
**When** the member calls buyer and seller entry routes
**Then** both access paths are authorized under the same member identity
**And** role checks do not require separate account contexts.

## Requirements Coverage

**Functional Requirements:** FR-003

- **FR-003**: Every verified member SHALL be able to act as both seller and buyer.

## Tasks / Subtasks

- [ ] Design unified member profile model (AC: #1)
  - [ ] Ensure member_profile supports both buyer and seller contexts
  - [ ] Add fields to track buyer activity (orders, cart)
  - [ ] Add fields to track seller activity (listings, seller_stats)
  - [ ] Avoid separate buyer/seller role flags (unified by default)

- [ ] Implement buyer route access for verified members (AC: #1)
  - [ ] Enable `/store/listings` (browse marketplace)
  - [ ] Enable `/store/cart` (add to cart)
  - [ ] Enable `/store/checkout` (initiate payment)
  - [ ] Enable `/store/orders` (view own orders as buyer)
  - [ ] Apply `requireMembership()` middleware to all buyer routes

- [ ] Implement seller route access for verified members (AC: #1)
  - [ ] Enable `/store/member/listings` (create/manage listings)
  - [ ] Enable `/store/member/listings/:id/publish` (publish listing)
  - [ ] Enable `/store/member/orders` (view orders for own listings)
  - [ ] Apply `requireMembership()` middleware to all seller routes

- [ ] Create member context helper for dual-role access (AC: #1)
  - [ ] Implement `getMemberContext(req)` helper
  - [ ] Return unified member profile with buyer and seller capabilities
  - [ ] Include member_id, wallet_address, verification_status
  - [ ] Include buyer context: active_cart_id, order_count
  - [ ] Include seller context: listing_count, seller_stats

- [ ] Update authorization logic to support dual roles (AC: #1)
  - [ ] Remove any buyer-only or seller-only role checks
  - [ ] Verify member can access both buyer and seller routes
  - [ ] Ensure no role conflict errors
  - [ ] Document that all verified members have both capabilities

- [ ] Link member_profile to Medusa customer for buyer flows (AC: #1)
  - [ ] Create module link: member_profile → customer
  - [ ] Ensure cart and order operations use customer context
  - [ ] Maintain member_id reference in customer metadata
  - [ ] Support member-to-customer resolution in queries

- [ ] Link member_profile to seller operations (AC: #1)
  - [ ] Ensure listing entity has seller_member_profile_id
  - [ ] Support seller-scoped queries by member_profile_id
  - [ ] Maintain seller attribution in order records
  - [ ] Enable seller dashboard queries

- [ ] Add member activity tracking (AC: #1)
  - [ ] Track last buyer activity timestamp
  - [ ] Track last seller activity timestamp
  - [ ] Track total orders as buyer
  - [ ] Track total listings as seller
  - [ ] Use for analytics and member engagement

- [ ] Write unit tests for dual-role access
  - [ ] Test member can create listing (seller action)
  - [ ] Test same member can browse marketplace (buyer action)
  - [ ] Test same member can add to cart (buyer action)
  - [ ] Test same member can view own orders as buyer
  - [ ] Test same member can view orders for own listings as seller

- [ ] Write integration tests for unified member flows
  - [ ] Test member creates listing → publishes → browses marketplace → buys from another member
  - [ ] Test member receives order for own listing while having active cart as buyer
  - [ ] Test member context is consistent across buyer and seller routes
  - [ ] Test no role conflict errors occur

## Dev Notes

### Architecture Patterns and Constraints

**Unified Member Model (CRITICAL):**
- Every verified member is BOTH buyer and seller by default
- No separate buyer/seller roles or flags
- Single member_profile entity supports both contexts
- Authorization is based on membership verification, not role assignment

**Member Context Pattern:**
```typescript
interface MemberContext {
  member_id: string;
  member_profile_id: string;
  wallet_address: string;
  verification_status: "verified" | "pending" | "failed";
  
  // Buyer context
  customer_id?: string; // Linked Medusa customer
  active_cart_id?: string;
  buyer_order_count: number;
  
  // Seller context
  seller_listing_count: number;
  seller_order_count: number;
  
  // Activity tracking
  last_buyer_activity_at?: Date;
  last_seller_activity_at?: Date;
}
```

**Route Access Pattern:**
- Buyer routes: `/store/listings`, `/store/cart`, `/store/checkout`, `/store/orders`
- Seller routes: `/store/member/listings`, `/store/member/orders`
- Both use same `requireMembership()` middleware
- No additional role checks needed

### Technical Requirements

**Member Profile Schema Extensions:**
- Add `customer_id` link to Medusa customer (for buyer flows)
- Add `buyer_order_count`, `seller_order_count` for tracking
- Add `last_buyer_activity_at`, `last_seller_activity_at` timestamps
- Keep schema flexible for future role extensions

**Module Links:**
- `member_profile` → Medusa `customer` (1:1 for buyer operations)
- `member_profile` → `listing` (1:many for seller operations)
- Use `query.graph()` for cross-module reads

**Customer Creation Flow:**
- When member first acts as buyer, create linked Medusa customer
- Store member_id in customer metadata
- Use customer for cart and order operations
- Maintain bidirectional lookup: member ↔ customer

**Seller Attribution:**
- All listings store `seller_member_profile_id`
- Orders store both buyer customer_id and seller member_profile_id
- Enable seller dashboard queries by member_profile_id

### Library and Framework Requirements

**Required Dependencies:**
- `@medusajs/medusa` (v2) - Core framework
- Membership module (from Story 1.1)
- Marketplace module (from Epic 2)

**No Additional Role Management:**
- Do NOT use separate role management libraries
- Do NOT implement RBAC for buyer/seller distinction
- Membership verification is the only access gate

### File Structure Requirements

**Member Context Helper:**
```
src/utils/member-context.ts
```

**Updated Member Profile Model:**
```
src/modules/membership/models/member-profile.ts
```

**Module Links Configuration:**
```
src/modules/membership/index.ts (define links)
```

### Testing Requirements

**Unit Test Coverage:**
- Member context helper returns correct buyer/seller data
- Member profile model supports both contexts
- Module links resolve correctly

**Integration Test Coverage:**
- Member creates listing (seller action)
- Same member browses marketplace (buyer action)
- Same member adds item to cart (buyer action)
- Same member completes purchase (buyer action)
- Same member views seller dashboard (seller action)
- No role conflicts or authorization errors

**Test Scenarios:**
1. **Dual Activity Flow:**
   - Member A creates and publishes listing
   - Member A browses marketplace
   - Member A adds Member B's listing to cart
   - Member A completes purchase
   - Member A views buyer orders
   - Member A views seller orders
   - All actions succeed with same member identity

2. **Concurrent Buyer/Seller:**
   - Member has active cart as buyer
   - Member receives order for own listing as seller
   - Both contexts remain valid and accessible

3. **First-Time Buyer:**
   - Verified member (no prior buyer activity)
   - Member browses marketplace
   - Customer entity is auto-created
   - Cart is initialized
   - Member can complete purchase

### Observability Requirements

**Structured Logging Fields:**
- `member_id` - Unified member identifier
- `action_type` - "buyer" or "seller"
- `route_path` - Route accessed
- `customer_id` - Linked customer (if buyer action)
- `listing_id` - Listing (if seller action)
- `correlation_id` - Request tracing

**Activity Events:**
- Event: `member.buyer_activity`
  - Payload: member_id, action (browse/cart/checkout), timestamp
- Event: `member.seller_activity`
  - Payload: member_id, action (create_listing/publish), timestamp

**Operational Metrics:**
- Members active as buyers (last 30 days)
- Members active as sellers (last 30 days)
- Members active as both (last 30 days)
- Average buyer orders per member
- Average seller listings per member

### Project Structure Notes

**Alignment with Unified Project Structure:**
- Extend membership module (don't create separate buyer/seller modules)
- Use module links for cross-module relationships
- Follow Medusa v2 customer/order patterns for buyer flows
- Use custom marketplace module for seller flows

**Avoid Anti-Patterns:**
- ❌ Don't create separate buyer and seller profiles
- ❌ Don't use role flags or enums for buyer/seller distinction
- ❌ Don't require role switching or context changes
- ✅ Use single member profile with dual capabilities
- ✅ Use module links for buyer (customer) and seller (listings) contexts

### Dependencies on Other Stories

**Depends on:**
- Story 1.1: Member verification and member_profile model
- Story 1.2: Member authentication middleware

**Enables:**
- Story 1.4: Seller ownership scope (uses seller context)
- Epic 2: Listing creation (uses seller context)
- Epic 3: Buyer marketplace (uses buyer context)
- Epic 4: Checkout (uses buyer context)

### References

**Source Documents:**
- [PRD: Section 2.1 - Community Member (Seller + Buyer)](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#21-community-member-seller--buyer)
- [PRD: FR-003](c:/Projects/Stalabard/_bmad-output/planning-artifacts/prd.md#51-membership-and-access-control)
- [Architecture: Section 1.2 - Functional Baseline (Every verified member can buy and sell)](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#12-functional-baseline-for-mvp)
- [Architecture: Section 3.1.A - Membership Module](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#a-membership-module)
- [Architecture: Section 3.3 - Module Links](c:/Projects/Stalabard/_bmad-output/planning-artifacts/architecture.md#33-module-links-cross-module)
- [Epic 1: Story 1.3](c:/Projects/Stalabard/_bmad-output/planning-artifacts/epics.md#story-13-enable-dual-buyerseller-capability-for-verified-members)

**External Documentation:**
- Medusa v2 Module Links: https://docs.medusajs.com/development/modules/module-links
- Medusa v2 Customer Entity: https://docs.medusajs.com/references/customer

## Dev Agent Record

### Agent Model Used

_To be filled by dev agent_

### Debug Log References

_To be filled by dev agent during implementation_

### Completion Notes List

_To be filled by dev agent upon completion_

### File List

_To be filled by dev agent with all created/modified files_
