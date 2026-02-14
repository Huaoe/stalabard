---
name: medusa-reference-module-links
description: Reference guide for module links in Medusa. Load when implementing relationships between modules and cross-module data access.
---

# Module Links Reference

Module links define relationships between entities in different modules while maintaining module isolation.

## When to Use Module Links

- Linking custom module entities to core Medusa entities (products, orders, customers)
- Creating relationships between custom modules
- Maintaining referential integrity across modules
- Enabling efficient cross-module queries

## Defining Module Links

Module links are defined in the module's `index.ts`:

```typescript
// src/modules/review/index.ts
import ReviewModuleService from "./service"
import { Module } from "@medusajs/framework/utils"

export const REVIEW_MODULE = "review"

export default Module(REVIEW_MODULE, {
  service: ReviewModuleService,
  links: [
    {
      alias: "product_reviews",
      foreignKey: "product_id",
      primaryKey: "id",
      foreignKeyTargetField: "id",
    },
  ],
})
```

## Module Link Configuration

```typescript
{
  alias: "product_reviews",           // Name of the relationship
  foreignKey: "product_id",           // Field in your module
  primaryKey: "id",                   // Field in target module
  foreignKeyTargetField: "id",        // Target module's primary key
}
```

## Accessing Linked Data

Use `query.graph()` to access linked data:

```typescript
// Get products with their reviews
const { data: products } = await query.graph(
  {
    entity: "product",
    fields: ["id", "title", "product_reviews.*"],
  },
  { container }
)

// Result includes reviews for each product
products.forEach(product => {
  console.log(`${product.title} has ${product.product_reviews.length} reviews`)
})
```

## Querying Through Links

Filter and query through module links:

```typescript
// Get reviews for a specific product
const { data: reviews } = await query.graph(
  {
    entity: "review",
    fields: ["id", "rating", "content"],
    filters: {
      product_id: productId,
    },
  },
  { container }
)
```

## Best Practices

- **Keep modules isolated**: Use links instead of direct service calls
- **Define links in module definition**: Not in data models
- **Use query.graph() for cross-module queries**: Respects module boundaries
- **Avoid circular dependencies**: Design links carefully
- **Use meaningful aliases**: Make relationships clear

## Example: Product Reviews Module

```typescript
// src/modules/review/models/review.ts
import { model } from "@medusajs/framework/utils"

const Review = model.define("review", {
  id: model.id().primaryKey(),
  product_id: model.text(),
  customer_id: model.text(),
  rating: model.integer(),
  content: model.text(),
})

export default Review
```

```typescript
// src/modules/review/service.ts
import { MedusaService } from "@medusajs/framework/utils"
import Review from "./models/review"

class ReviewModuleService extends MedusaService({
  Review,
}) {}

export default ReviewModuleService
```

```typescript
// src/modules/review/index.ts
import ReviewModuleService from "./service"
import { Module } from "@medusajs/framework/utils"

export const REVIEW_MODULE = "review"

export default Module(REVIEW_MODULE, {
  service: ReviewModuleService,
  links: [
    {
      alias: "product_reviews",
      foreignKey: "product_id",
      primaryKey: "id",
      foreignKeyTargetField: "id",
    },
  ],
})
```

## Implementation Checklist

- [ ] Define module links in module definition
- [ ] Use meaningful alias names
- [ ] Ensure foreign key fields exist in data model
- [ ] Register module in medusa-config.ts
- [ ] Generate and run migrations
- [ ] Test cross-module queries with query.graph()
- [ ] Verify data relationships work correctly
