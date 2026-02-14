---
name: medusa-reference-authentication
description: Reference guide for authentication in Medusa. Load when implementing protected routes, user authentication, and authorization.
---

# Authentication Reference

## Store Routes (Customer Authentication)

Store routes use customer authentication via session or bearer token:

```typescript
// api/store/my-route/middlewares.ts
import { authenticate } from "@medusajs/framework"
import { validateAndTransformBody } from "@medusajs/framework"

export const myStoreMiddlewares = [
  {
    matcher: "/store/my-route",
    method: "POST",
    middlewares: [
      authenticate("customer", ["session", "bearer"]),
      validateAndTransformBody(CreateMySchema),
    ],
  },
]
```

## Admin Routes (User Authentication)

Admin routes use user authentication (staff/admin):

```typescript
// api/admin/my-route/middlewares.ts
import { authenticate } from "@medusajs/framework"
import { validateAndTransformBody } from "@medusajs/framework"

export const myAdminMiddlewares = [
  {
    matcher: "/admin/my-route",
    method: "POST",
    middlewares: [
      authenticate("user", ["session", "bearer"]),
      validateAndTransformBody(CreateMySchema),
    ],
  },
]
```

## Accessing Authenticated User

Use `AuthenticatedMedusaRequest` to access the authenticated user:

```typescript
// api/admin/my-route/route.ts
import { AuthenticatedMedusaRequest, MedusaResponse } from "@medusajs/framework/http"
import { CreateMySchema } from "./middlewares"

export async function POST(
  req: AuthenticatedMedusaRequest<CreateMySchema>,
  res: MedusaResponse
) {
  // Access authenticated user
  const userId = req.auth_entity.id
  const userEmail = req.auth_entity.email
  
  // Use in workflow
  const result = await myWorkflow(req.scope).run({
    input: {
      ...req.validatedBody,
      created_by: userId,
    },
  })

  res.json({ data: result })
}
```

## Authentication Methods

- **session**: Cookie-based authentication
- **bearer**: Bearer token in Authorization header
- **api_token**: API token authentication

```typescript
// Accept multiple auth methods
authenticate("customer", ["session", "bearer"])
authenticate("user", ["session", "bearer"])
```

## Public Routes (No Authentication)

Routes without the `authenticate` middleware are public:

```typescript
// api/store/public-route/middlewares.ts
export const publicMiddlewares = [
  {
    matcher: "/store/public-route",
    method: "GET",
    middlewares: [
      // No authenticate middleware - this route is public
    ],
  },
]
```

## Authorization Checks

Perform authorization checks in workflows, not routes:

```typescript
// src/workflows/steps/update-item.ts
import { createStep, StepResponse } from "@medusajs/framework/workflows-sdk"

export const updateItemStep = createStep(
  "update-item",
  async (input: { id: string; user_id: string; data: any }, { container }) => {
    const itemService = container.resolve("item")
    
    // Check ownership
    const item = await itemService.retrieveItem(input.id)
    if (item.owner_id !== input.user_id) {
      throw new Error("Not authorized to update this item")
    }
    
    const [updated] = await itemService.updateItems(input.id, input.data)
    return new StepResponse(updated, updated.id)
  }
)
```

## Implementation Checklist

- [ ] Determine if route should be protected (store vs admin)
- [ ] Add appropriate authenticate middleware
- [ ] Use AuthenticatedMedusaRequest type for protected routes
- [ ] Access authenticated user via req.auth_entity
- [ ] Perform authorization checks in workflows
- [ ] Test with and without authentication
