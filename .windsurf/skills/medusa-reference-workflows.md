---
name: medusa-reference-workflows
description: Reference guide for creating workflows in Medusa. Load when implementing mutations, business logic, and workflow steps.
---

# Creating Workflows Reference

Workflows are the standard way to perform mutations (create, update, delete) in modules in Medusa. If you have built a custom module and need to perform mutations on models in the module, you should create a workflow.

## Basic Workflow Structure

**File Organization:**
- **Recommended**: Create workflow steps in `src/workflows/steps/[step-name].ts`
- Workflow composition functions go in `src/workflows/[workflow-name].ts`
- This keeps steps reusable and organized

```typescript
// src/workflows/steps/create-my-model.ts
import { createStep, StepResponse } from "@medusajs/framework/workflows-sdk"

type Input = {
  my_key: string
}

// Note: a step should only do one mutation this ensures rollback mechanisms work
// For workflows that retry build your steps to be idempotent
export const createMyModelStep = createStep(
  "create-my-model",
  async (input: Input, { container }) => {
    const myModule = container.resolve("my")

    const [newMy] = await myModule.createMyModels({
      ...input,
    })

    return new StepResponse(
      newMy,
      newMy.id // explicit compensation input - otherwise defaults to step's output
    )
  },
  // Optional compensation function
  async (id, { container }) => {
    const myModule = container.resolve("my")
    await myModule.deleteMyModels(id)
  }
)

// src/workflows/create-my-model.ts
import { createWorkflow, WorkflowResponse } from "@medusajs/framework/workflows-sdk"
import { createMyModelStep } from "./steps/create-my-model"

type Input = {
  my_key: string
}

const createMyModel = createWorkflow(
  "create-my-model",
  // Note: See "Workflow Composition Rules" section below for important constraints
  // The workflow function must be a regular synchronous function (not async/arrow)
  // No direct variable manipulation, conditionals, or date creation - use transform/when instead
  function (input: Input) {
    const newMy = createMyModelStep(input)

    return new WorkflowResponse({
      newMy,
    })
  }
)

export default createMyModel
```

## Workflow Composition Rules

The workflow composition function runs at application load time and has important limitations:

### Function Declaration
- ✅ Use regular synchronous functions
- ❌ No `async` functions
- ❌ No arrow functions (use `function` keyword)

### Using Steps Multiple Times

**⚠️ CRITICAL**: When using the same step multiple times in a workflow, you MUST rename each invocation AFTER the first invocation using `.config()` to avoid conflicts.

```typescript
// ✅ CORRECT - Rename each step invocation with .config()
export const processCustomersWorkflow = createWorkflow(
  "process-customers",
  function (input) {
    const customers = transform({ ids: input.customer_ids }, (input) => input.ids)

    // First invocation - no need to rename
    const customer1 = fetchCustomerStep(customers[0])

    // Second invocation - different name
    const customer2 = fetchCustomerStep(customers[1]).config({
      name: "fetch-customer-2"
    })

    const result = transform({ customer1, customer2 }, (data) => ({
      customers: [data.customer1, data.customer2]
    }))

    return new WorkflowResponse(result)
  }
)

// ❌ WRONG - Calling the same step multiple times without renaming
export const processCustomersWorkflow = createWorkflow(
  "process-customers",
  function (input) {
    const customers = transform({ ids: input.customer_ids }, (input) => input.ids)

    // This will cause runtime errors - duplicate step names
    const customer1 = fetchCustomerStep(customers[0])
    const customer2 = fetchCustomerStep(customers[1]) // ❌ Conflict!

    return new WorkflowResponse({ customers: [customer1, customer2] })
  }
)
```

**Why this matters:**
- Medusa uses step names to track execution state
- Duplicate names cause conflicts in the workflow execution engine
- Each step invocation needs a unique identifier
- The workflow will fail at runtime if steps aren't renamed

### Variable Operations
- ❌ No direct variable manipulation or concatenation → Use `transform({ in }, ({ in }) => \`Transformed: ${in}\`)` instead
- Variables lack values until execution time - all operations must use `transform()`

### Date/Time Operations
- ❌ No `new Date()` (will be fixed to load time) → Wrap in `transform()` instead
- Use `transform()` for any dynamic values

### Conditionals
- ❌ No `if` statements in composition function
- ✅ Use `when()` to conditionally execute steps

```typescript
// ❌ WRONG
function (input) {
  if (input.notify) {
    const result = sendEmailStep(input)
  }
  return new WorkflowResponse({})
}

// ✅ CORRECT
function (input) {
  const shouldNotify = when({ notify: input.notify }, ({ notify }) => notify)
  const result = sendEmailStep(input).when(shouldNotify)
  return new WorkflowResponse({})
}
```

## Step Input and Output

Each step receives input and returns output via `StepResponse`:

```typescript
export const myStep = createStep(
  "my-step",
  async (input: MyInput, { container }) => {
    // Do work
    const result = await doSomething(input)
    
    // Return result and compensation input
    return new StepResponse(
      result, // This becomes the step's output
      result.id // This is passed to compensation function
    )
  },
  async (compensationInput, { container }) => {
    // Rollback logic
    await undoSomething(compensationInput)
  }
)
```

## Compensation Functions

Compensation functions handle rollback if a workflow fails:

```typescript
export const createOrderStep = createStep(
  "create-order",
  async (input, { container }) => {
    const orderService = container.resolve("order")
    const [order] = await orderService.createOrders(input)
    
    return new StepResponse(order, order.id)
  },
  // Compensation: delete the order if workflow fails
  async (orderId, { container }) => {
    const orderService = container.resolve("order")
    await orderService.deleteOrders(orderId)
  }
)
```

## Idempotency

Build steps to be idempotent so they can be safely retried:

```typescript
// ✅ GOOD - Idempotent (safe to retry)
export const createUniqueItemStep = createStep(
  "create-unique-item",
  async (input, { container }) => {
    const itemService = container.resolve("item")
    
    // Check if already exists
    const existing = await itemService.listItems({
      filters: { external_id: input.external_id }
    })
    
    if (existing.length > 0) {
      return new StepResponse(existing[0], existing[0].id)
    }
    
    const [item] = await itemService.createItems(input)
    return new StepResponse(item, item.id)
  }
)

// ❌ BAD - Not idempotent (creates duplicates on retry)
export const createItemStep = createStep(
  "create-item",
  async (input, { container }) => {
    const itemService = container.resolve("item")
    const [item] = await itemService.createItems(input)
    return new StepResponse(item, item.id)
  }
)
```

## Executing Workflows

Execute workflows from API routes or other workflows:

```typescript
// In an API route
import createMyModel from "@/workflows/create-my-model"

export async function POST(req: MedusaRequest, res: MedusaResponse) {
  const { result } = await createMyModel(req.scope)
    .run({
      input: {
        my_key: req.validatedBody.my_key,
      },
    })

  res.json({ data: result })
}
```

## Implementation Checklist

- [ ] Define the input type for your workflow
- [ ] Create step function (one mutation per step)
- [ ] Add compensation function to steps for rollback
- [ ] Create workflow composition function
- [ ] Follow workflow composition rules (no async, no arrow functions, etc.)
- [ ] Return WorkflowResponse with results
- [ ] Test idempotency (workflow can be retried safely)
- [ ] **CRITICAL:** Run build to validate: `npx medusa build`
