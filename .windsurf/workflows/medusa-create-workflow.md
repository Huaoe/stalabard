---
description: Create a new Medusa workflow for handling mutations and business logic
---

# Create Medusa Workflow

This workflow guides you through creating a complete workflow in Medusa.

## Prerequisites

- Custom module created (or using existing module)
- Understanding of workflow structure (see medusa-reference-workflows skill)

## Steps

1. **Load the medusa-reference-workflows skill**
   - This provides detailed guidance on workflow composition rules and patterns

2. **Define the input type**
   - Create a TypeScript type for workflow input
   - Include all required parameters

3. **Create workflow steps**
   - Create `src/workflows/steps/[step-name].ts` for each mutation
   - Each step should do ONE mutation only
   - Include compensation function for rollback

4. **Create workflow composition**
   - Create `src/workflows/[workflow-name].ts`
   - Use regular `function` keyword (not async/arrow)
   - Follow composition rules (no direct variable manipulation, use `transform()`)

5. **Handle step dependencies**
   - If using same step multiple times, rename with `.config()`
   - Use `transform()` for variable operations
   - Use `when()` for conditional execution

6. **Return WorkflowResponse**
   - Include all results needed by caller
   - Ensure proper typing

7. **Build and validate**
   - Run: `npx medusa build`
   - Catches composition rule violations

8. **Create API route to execute workflow**
   - Create route in `api/[scope]/[feature]/route.ts`
   - Import workflow at top level (not dynamic)
   - Execute with: `await workflow(req.scope).run({ input })`

## Validation Checklist

- [ ] Input type properly defined
- [ ] Each step does only one mutation
- [ ] Compensation functions handle rollback
- [ ] Workflow uses regular function (not async/arrow)
- [ ] No direct variable manipulation (use transform())
- [ ] No conditionals (use when())
- [ ] Duplicate steps renamed with .config()
- [ ] Build completes without errors
- [ ] Workflow executes successfully from API route

## Common Issues

**"Workflow composition function must be synchronous"**
- Remove `async` keyword from workflow function
- Use regular `function` keyword instead of arrow function

**"Duplicate step name" error**
- When using same step multiple times, rename with `.config()`
- Example: `myStep(input).config({ name: "my-step-2" })`

**Variables not accessible in composition**
- Use `transform()` for all variable operations
- Don't try to manipulate variables directly

**Compensation not working**
- Ensure compensation function is defined
- Compensation input should match what you return from StepResponse
- Test rollback by intentionally failing a step

## Workflow Composition Rules

**DO:**
- Use regular synchronous functions
- Use `transform()` for variable operations
- Use `when()` for conditionals
- Rename duplicate step invocations
- Return WorkflowResponse with results

**DON'T:**
- Use async functions
- Use arrow functions
- Manipulate variables directly
- Use if/else statements
- Create new Date() in composition
- Use dynamic imports
