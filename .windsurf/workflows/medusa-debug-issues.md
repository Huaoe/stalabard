---
description: Debug and troubleshoot common Medusa development issues
---

# Debug Medusa Issues

This workflow helps you diagnose and fix common Medusa development problems.

## Identifying the Issue

### 1. Module Not Found at Runtime

**Symptoms:**
- "Cannot resolve module" error
- Module service not available in container

**Diagnosis:**
- Check module name is camelCase (no dashes)
- Verify module is registered in medusa-config.ts
- Ensure migrations were run

**Fix:**
```bash
# Verify module registration
grep -r "resolve\(\"[module-name]\"\)" src/

# Check medusa-config.ts has the module
cat medusa-config.ts | grep modules

# Regenerate migrations if needed
npx medusa db:generate [module-name]
npx medusa db:migrate
```

### 2. Database Tables Don't Exist

**Symptoms:**
- "Table not found" database errors
- Queries return empty results

**Diagnosis:**
- Migrations not generated
- Migrations not run
- Wrong module name in migration

**Fix:**
```bash
# Generate migrations
npx medusa db:generate [module-name]

# Run migrations
npx medusa db:migrate

# Check migration status
npx medusa db:migrate --status
```

### 3. Workflow Composition Errors

**Symptoms:**
- "Workflow composition function must be synchronous"
- "Duplicate step name" error
- Variables not accessible

**Diagnosis:**
- Using async/arrow functions in composition
- Same step used multiple times without renaming
- Direct variable manipulation instead of transform()

**Fix:**
```typescript
// ✅ CORRECT
function (input) {
  const step1 = myStep(input)
  const step2 = myStep(input).config({ name: "my-step-2" })
  
  const result = transform({ step1, step2 }, (data) => ({
    results: [data.step1, data.step2]
  }))
  
  return new WorkflowResponse(result)
}

// ❌ WRONG - async, no rename, direct manipulation
async (input) => {
  const step1 = myStep(input)
  const step2 = myStep(input)  // Duplicate!
  const combined = step1 + step2  // Direct manipulation!
  return new WorkflowResponse(combined)
}
```

### 4. Type Errors in Routes

**Symptoms:**
- "req.validatedBody is not typed"
- TypeScript errors when accessing request body

**Diagnosis:**
- Schema type not passed to MedusaRequest<T>
- Schema type not exported from middlewares

**Fix:**
```typescript
// api/[scope]/[feature]/middlewares.ts
export const MySchema = z.object({ /* ... */ })
export type MySchema = z.infer<typeof MySchema>  // Export type!

// api/[scope]/[feature]/route.ts
export async function POST(
  req: MedusaRequest<MySchema>,  // Pass type!
  res: MedusaResponse
) {
  const { field } = req.validatedBody  // Now typed!
}
```

### 5. Middleware Not Working

**Symptoms:**
- Validation not running
- Request body not validated
- Middleware not applied

**Diagnosis:**
- Middleware not registered in api/middlewares.ts
- Middleware path doesn't match route
- Middleware array not spread correctly

**Fix:**
```typescript
// api/middlewares.ts
import { defineMiddlewares } from "@medusajs/framework/http"
import { myMiddlewares } from "./store/[feature]/middlewares"

export default defineMiddlewares({
  routes: [...myMiddlewares],  // Spread the array!
})
```

### 6. Build Failures

**Symptoms:**
- `npx medusa build` fails
- Type errors after changes
- Runtime errors

**Diagnosis:**
- Type mismatches
- Missing imports
- Configuration errors

**Fix:**
```bash
# Run build with verbose output
npx medusa build --verbose

# Check for type errors
npx tsc --noEmit

# Clear build cache
rm -rf .medusa/build
npx medusa build
```

## Debugging Steps

1. **Check logs**
   ```bash
   # Development server logs
   npm run dev
   
   # Check for error messages
   # Look for stack traces and error codes
   ```

2. **Verify configuration**
   ```bash
   # Check medusa-config.ts
   cat medusa-config.ts
   
   # Verify environment variables
   cat .env
   ```

3. **Test database**
   ```bash
   # Connect to database
   psql $DATABASE_URL
   
   # List tables
   \dt
   
   # Check schema
   \d [table-name]
   ```

4. **Test API endpoints**
   ```bash
   # Test route
   curl -X POST http://localhost:9000/store/my-route \
     -H "Content-Type: application/json" \
     -d '{"key": "value"}'
   ```

5. **Check module resolution**
   ```typescript
   // In a route or workflow
   const module = container.resolve("[module-name]")
   console.log("Module resolved:", module)
   ```

## Common Error Messages

| Error | Cause | Solution |
|-------|-------|----------|
| "Cannot resolve module" | Module not registered | Add to medusa-config.ts |
| "Table not found" | Migrations not run | Run `npx medusa db:migrate` |
| "Duplicate step name" | Same step used twice | Use `.config({ name: "..." })` |
| "Workflow must be synchronous" | Using async/arrow function | Use regular `function` keyword |
| "req.validatedBody is not typed" | Missing schema type | Pass type to `MedusaRequest<T>` |

## Validation Checklist

- [ ] Check error message and stack trace
- [ ] Verify configuration files
- [ ] Run `npx medusa build`
- [ ] Check database migrations
- [ ] Test with curl/Postman
- [ ] Review recent code changes
- [ ] Check Medusa documentation
- [ ] Ask for help if stuck
