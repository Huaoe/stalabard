---
name: medusa-reference-api-routes
description: Reference guide for creating API routes in Medusa. Load when implementing HTTP endpoints, validation, and authentication.
---

# Custom API Routes Reference

API routes (also called "endpoints") are the primary way to expose custom functionality to storefronts and admin dashboards.

## Path Conventions

### Store API Routes (Storefront)
- **Path prefix**: `/store/<rest-of-path>`
- **Examples**: `/store/newsletter-signup`, `/store/custom-search`
- **Authentication**: SDK automatically includes publishable API key

### Admin API Routes (Dashboard)
- **Path prefix**: `/admin/<rest-of-path>`
- **Examples**: `/admin/custom-reports`, `/admin/bulk-operations`
- **Authentication**: SDK automatically includes auth headers (bearer/session)

**Detailed authentication patterns**: See medusa-reference-authentication skill

## Middleware Validation

**⚠️ CRITICAL**: Always validate request bodies using Zod schemas and the `validateAndTransformBody` middleware.

### Combining Multiple Middlewares

When you need both authentication AND validation, pass them as an array. **NEVER nest validation inside authenticate:**

```typescript
// ✅ CORRECT - Multiple middlewares in array
export default defineMiddlewares({
  routes: [
    {
      matcher: "/store/products/:id/reviews",
      method: "POST",
      middlewares: [
        authenticate("customer", ["session", "bearer"]),
        validateAndTransformBody(CreateReviewSchema)
      ],
    },
  ],
})

// ❌ WRONG - Don't nest validator inside authenticate
export default defineMiddlewares({
  routes: [
    {
      matcher: "/store/products/:id/reviews",
      method: "POST",
      middlewares: [authenticate("customer", ["session", "bearer"], {
        validator: CreateReviewSchema  // This doesn't work!
      })],
    },
  ],
})
```

**Middleware order matters:** Put `authenticate` before `validateAndTransformBody` so authentication happens first.

### Step 1: Create Middleware File

```typescript
// api/store/[feature]/middlewares.ts
import { MiddlewareRoute, validateAndTransformBody } from "@medusajs/framework"
import { z } from "zod"

export const CreateMySchema = z.object({
  email: z.string().email(),
  name: z.string().min(2),
  // other fields
})

// Export the inferred type for use in route handlers
export type CreateMySchema = z.infer<typeof CreateMySchema>

export const myMiddlewares: MiddlewareRoute[] = [
  {
    matcher: "/store/my-route",
    method: "POST",
    middlewares: [validateAndTransformBody(CreateMySchema)],
  },
]
```

### Step 2: Register in api/middlewares.ts

```typescript
// api/middlewares.ts
import { defineMiddlewares } from "@medusajs/framework/http"
import { myMiddlewares } from "./store/[feature]/middlewares"

export default defineMiddlewares({
  routes: [...myMiddlewares],
})
```

**⚠️ CRITICAL - Middleware Export Pattern:**

Middlewares are exported as **named arrays**, NOT default exports with config objects:

```typescript
// ✅ CORRECT - Named export of MiddlewareRoute array
// api/store/reviews/middlewares.ts
export const reviewMiddlewares: MiddlewareRoute[] = [
  {
    matcher: "/store/reviews",
    method: "POST",
    middlewares: [validateAndTransformBody(CreateReviewSchema)],
  },
]

// ✅ CORRECT - Import and spread the named array
// api/middlewares.ts
import { reviewMiddlewares } from "./store/reviews/middlewares"

export default defineMiddlewares({
  routes: [...reviewMiddlewares],
})
```

```typescript
// ❌ WRONG - Don't use default export with .config
// api/store/reviews/middlewares.ts
export default {
  config: {
    routes: [...], // This is NOT the middleware pattern!
  },
}

// ❌ WRONG - Don't access .config.routes
// api/middlewares.ts
import reviewMiddlewares from "./store/reviews/middlewares"
export default defineMiddlewares({
  routes: [...reviewMiddlewares.config.routes], // This doesn't work!
})
```

**Why this matters:**
- Middleware files export arrays directly, not config objects
- Route files (like `route.ts`) use `export const config = defineRouteConfig(...)`
- Don't confuse the two patterns - middlewares are simpler (just an array)

### Step 3: Use Typed req.validatedBody in Route

**⚠️ CRITICAL**: When using `req.validatedBody`, you MUST pass the inferred Zod schema type as a type argument to `MedusaRequest`. Otherwise, you'll get TypeScript errors when accessing `req.validatedBody`.

```typescript
// api/store/my-route/route.ts
import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"
import { CreateMySchema } from "./middlewares"

// ✅ CORRECT: Pass the Zod schema type as type argument
export async function POST(
  req: MedusaRequest<CreateMySchema>,
  res: MedusaResponse
) {
  // Now req.validatedBody is properly typed
  const { email, name } = req.validatedBody
  
  // ... handle request
}

// ❌ WRONG: No type argument - req.validatedBody will be any
export async function POST(
  req: MedusaRequest,
  res: MedusaResponse
) {
  // req.validatedBody is not typed
  const { email, name } = req.validatedBody // TypeScript error!
}
```

## Basic Route Structure

```typescript
// api/store/my-route/route.ts
import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"
import { CreateMySchema } from "./middlewares"

export async function POST(
  req: MedusaRequest<CreateMySchema>,
  res: MedusaResponse
) {
  const { email, name } = req.validatedBody
  
  // Resolve services from container
  const myService = req.scope.resolve("my")
  
  // Execute business logic (via workflow)
  const result = await myWorkflow(req.scope).run({
    input: { email, name },
  })
  
  res.json({ data: result })
}

export async function GET(
  req: MedusaRequest,
  res: MedusaResponse
) {
  const myService = req.scope.resolve("my")
  const items = await myService.listMyItems()
  
  res.json({ data: items })
}
```

## Error Handling

Use Medusa's error classes for consistent error responses:

```typescript
import { MedusaError } from "@medusajs/framework/utils"

export async function POST(
  req: MedusaRequest<CreateMySchema>,
  res: MedusaResponse
) {
  try {
    const myService = req.scope.resolve("my")
    const result = await myService.createMyItem(req.validatedBody)
    res.json({ data: result })
  } catch (error) {
    if (error instanceof MedusaError) {
      res.status(error.code || 400).json({ message: error.message })
    } else {
      res.status(500).json({ message: "Internal server error" })
    }
  }
}
```

## Protected Routes

Use the `authenticate` middleware for protected routes:

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

## Using Workflows in API Routes

Always use workflows for mutations:

```typescript
// api/store/items/route.ts
import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"
import createItemWorkflow from "@/workflows/create-item"
import { CreateItemSchema } from "./middlewares"

export async function POST(
  req: MedusaRequest<CreateItemSchema>,
  res: MedusaResponse
) {
  const { result } = await createItemWorkflow(req.scope).run({
    input: req.validatedBody,
  })

  res.json({ data: result })
}
```

## Query Parameters

Access query parameters from the request:

```typescript
export async function GET(
  req: MedusaRequest,
  res: MedusaResponse
) {
  const { limit = 10, offset = 0 } = req.query
  
  const myService = req.scope.resolve("my")
  const items = await myService.listMyItems({
    take: parseInt(limit as string),
    skip: parseInt(offset as string),
  })
  
  res.json({ data: items })
}
```

## Implementation Checklist

- [ ] Create middleware file with Zod schema
- [ ] Export schema type from middlewares
- [ ] Register middlewares in api/middlewares.ts
- [ ] Create route handler with proper types
- [ ] Use workflows for mutations
- [ ] Handle errors appropriately
- [ ] Test with curl or Postman
- [ ] **CRITICAL:** Run build to validate: `npx medusa build`
