---
description: Create a new Medusa API route with validation and authentication
---

# Create Medusa API Route

This workflow guides you through creating a complete API route in Medusa.

## Prerequisites

- Understanding of API route structure (see medusa-reference-api-routes skill)
- Workflow created (for mutations) or module service ready (for queries)

## Steps

1. **Load the medusa-reference-api-routes skill**
   - This provides detailed guidance on route structure and validation patterns

2. **Determine route scope and path**
   - Store routes: `/store/[feature]/[action]`
   - Admin routes: `/admin/[feature]/[action]`
   - Example: `/store/reviews/create` or `/admin/reports/generate`

3. **Create middleware file (if POST/mutation)**
   - Create `api/[scope]/[feature]/middlewares.ts`
   - Define Zod schema for request validation
   - Export schema type for use in route
   - Define MiddlewareRoute array

4. **Register middlewares (if POST/mutation)**
   - Add to `api/middlewares.ts`
   - Import and spread middleware array
   - Ensure proper middleware order (authenticate before validate)

5. **Create route handler**
   - Create `api/[scope]/[feature]/route.ts`
   - Implement GET, POST, DELETE methods as needed
   - Use proper request types (MedusaRequest or AuthenticatedMedusaRequest)
   - Pass Zod schema type to MedusaRequest<T>

6. **Resolve services/workflows**
   - For queries: `const service = req.scope.resolve("[module]")`
   - For mutations: Import workflow at top, execute with `workflow(req.scope).run()`

7. **Handle errors**
   - Use try/catch blocks
   - Return appropriate HTTP status codes
   - Include error messages in response

8. **Build and test**
   - Run: `npx medusa build`
   - Test with curl or Postman
   - Verify request validation works
   - Verify authentication (if protected)

## Validation Checklist

- [ ] Route path follows conventions (/store or /admin)
- [ ] Zod schema defined for POST/mutation routes
- [ ] Schema type exported from middlewares
- [ ] Middlewares registered in api/middlewares.ts
- [ ] Route uses correct request type (MedusaRequest<T>)
- [ ] Workflow imported at top level (not dynamic)
- [ ] Error handling implemented
- [ ] Build completes without errors
- [ ] Route responds correctly to requests

## Common Issues

**"req.validatedBody is not typed"**
- Pass Zod schema type to MedusaRequest: `MedusaRequest<CreateSchema>`
- Export schema type from middlewares: `export type CreateSchema = z.infer<typeof CreateSchema>`

**"Middleware not working"**
- Ensure middleware is registered in api/middlewares.ts
- Check middleware path matches route matcher
- Verify middleware array is spread correctly

**"Workflow not executing"**
- Import workflow at top of file (not dynamic)
- Execute with: `await workflow(req.scope).run({ input })`
- Ensure workflow is properly exported

**"Authentication not working"**
- Add authenticate middleware before validate middleware
- Use AuthenticatedMedusaRequest for protected routes
- Access user with: `req.auth_entity.id`

## Route Structure Template

```typescript
// api/store/[feature]/middlewares.ts
import { validateAndTransformBody } from "@medusajs/framework"
import { z } from "zod"

export const CreateSchema = z.object({
  name: z.string(),
  // ... other fields
})

export type CreateSchema = z.infer<typeof CreateSchema>

export const featureMiddlewares = [
  {
    matcher: "/store/[feature]",
    method: "POST",
    middlewares: [validateAndTransformBody(CreateSchema)],
  },
]
```

```typescript
// api/store/[feature]/route.ts
import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"
import { CreateSchema } from "./middlewares"
import myWorkflow from "@/workflows/my-workflow"

export async function POST(
  req: MedusaRequest<CreateSchema>,
  res: MedusaResponse
) {
  const { result } = await myWorkflow(req.scope).run({
    input: req.validatedBody,
  })

  res.json({ data: result })
}

export async function GET(
  req: MedusaRequest,
  res: MedusaResponse
) {
  const service = req.scope.resolve("[module]")
  const items = await service.listItems()

  res.json({ data: items })
}
```
