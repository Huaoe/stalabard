---
name: medusa-reference-querying-data
description: Reference guide for querying data in Medusa. Load when implementing data retrieval, filtering, and cross-module queries.
---

# Querying Data Reference

## Basic Queries

Use the module service methods to query data:

```typescript
// List all items
const items = await itemService.listItems()

// Retrieve single item
const item = await itemService.retrieveItem(itemId)

// List with pagination
const items = await itemService.listItems({
  take: 10,
  skip: 0,
})

// List with filtering
const items = await itemService.listItems({
  filters: {
    status: "active",
    price: { $gte: 100 },
  },
})
```

## Cross-Module Queries with query.graph()

Use `query.graph()` to query across modules:

```typescript
// In a workflow step or API route
const { data: orders } = await query.graph(
  {
    entity: "order",
    fields: ["id", "total", "customer_id", "items.*"],
  },
  { container }
)

// With filtering
const { data: orders } = await query.graph(
  {
    entity: "order",
    fields: ["id", "total", "customer_id"],
    filters: {
      status: "completed",
    },
  },
  { container }
)
```

## Querying with Relations

Load related data using relations:

```typescript
const item = await itemService.retrieveItem(itemId, {
  relations: ["category", "reviews"],
})
```

## Query Configuration

Use `req.queryConfig` in API routes for flexible querying:

```typescript
// api/store/items/route.ts
import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"

export async function GET(
  req: MedusaRequest,
  res: MedusaResponse
) {
  const itemService = req.scope.resolve("item")
  
  // req.queryConfig automatically handles:
  // - fields parameter
  // - relations parameter
  // - pagination (limit, offset)
  const items = await itemService.listItems(req.queryConfig)
  
  res.json({ data: items })
}
```

**⚠️ CRITICAL**: Don't set explicit `fields` when using `req.queryConfig` - it's already handled:

```typescript
// ❌ WRONG - Don't override fields
const items = await itemService.listItems({
  ...req.queryConfig,
  fields: ["id", "name"], // Don't do this!
})

// ✅ CORRECT - Use queryConfig as-is
const items = await itemService.listItems(req.queryConfig)
```

## Soft Deletes

By default, soft-deleted records are excluded:

```typescript
// Get only active items (excludes soft-deleted)
const items = await itemService.listItems()

// Include soft-deleted items
const items = await itemService.listItems({
  withDeleted: true,
})
```

## Filtering Syntax

Common filtering patterns:

```typescript
// Exact match
filters: { status: "active" }

// Greater than / Less than
filters: { price: { $gte: 100, $lte: 500 } }

// In array
filters: { status: ["active", "pending"] }

// Not equal
filters: { status: { $ne: "deleted" } }

// Like (text search)
filters: { name: { $like: "%search%" } }

// Null checks
filters: { deleted_at: { $null: true } }
```

## Pagination

Use `take` and `skip` for pagination:

```typescript
const pageSize = 10
const pageNumber = 1

const items = await itemService.listItems({
  take: pageSize,
  skip: (pageNumber - 1) * pageSize,
})
```

## Counting

Get count of items:

```typescript
const [items, count] = await itemService.listAndCountItems({
  filters: { status: "active" },
})

console.log(`Found ${count} active items`)
```

## Implementation Checklist

- [ ] Use module service methods for basic queries
- [ ] Use query.graph() for cross-module queries
- [ ] Load relations when needed
- [ ] Use req.queryConfig in API routes
- [ ] Handle pagination with take/skip
- [ ] Apply appropriate filters
- [ ] Consider soft-deleted records
- [ ] Test query performance
