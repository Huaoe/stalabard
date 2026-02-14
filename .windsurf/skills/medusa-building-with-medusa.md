---
name: medusa-building-with-medusa
description: Load automatically when planning, researching, or implementing ANY Medusa backend features (custom modules, API routes, workflows, data models, module links, business logic). REQUIRED for all Medusa backend work in ALL modes (planning, implementation, exploration). Contains architectural patterns, best practices, and critical rules that MCP servers don't provide.
---

# Medusa Backend Development

Comprehensive backend development guide for Medusa applications. Contains patterns across 6 categories covering architecture, type safety, business logic placement, and common pitfalls.

## When to Apply

**Load this skill for ANY backend development task, including:**
- Creating or modifying custom modules and data models
- Implementing workflows for mutations
- Building API routes (store or admin)
- Defining module links between entities
- Writing business logic or validation
- Querying data across modules
- Implementing authentication/authorization

**Also load these skills when:**
- **medusa-building-admin-dashboard-customizations:** Building admin UI (widgets, pages, forms)
- **medusa-building-storefronts:** Calling backend API routes from storefronts (SDK integration)

## CRITICAL: Load Reference Files When Needed

**The quick reference below is NOT sufficient for implementation.** You MUST load relevant reference files before writing code for that component.

**Load these references based on what you're implementing:**

- **Creating a module?** → MUST load `medusa-reference-custom-modules` skill first
- **Creating workflows?** → MUST load `medusa-reference-workflows` skill first
- **Creating API routes?** → MUST load `medusa-reference-api-routes` skill first
- **Creating module links?** → MUST load `medusa-reference-module-links` skill first
- **Querying data?** → MUST load `medusa-reference-querying-data` skill first
- **Adding authentication?** → MUST load `medusa-reference-authentication` skill first

**Minimum requirement:** Load at least 1-2 reference skills relevant to your specific task before implementing.

## Critical Architecture Pattern

**ALWAYS follow this flow - never bypass layers:**

```
Module (data models + CRUD operations)
  ↓ used by
Workflow (business logic + mutations with rollback)
  ↓ executed by
API Route (HTTP interface, validation middleware)
  ↓ called by
Frontend (admin dashboard/storefront via SDK)
```

**Key conventions:**
- Only GET, POST, DELETE methods (never PUT/PATCH)
- Workflows are required for ALL mutations
- Business logic belongs in workflow steps, NOT routes
- Query with `query.graph()` for cross-module data retrieval
- Query with `query.index()` (Index Module) for filtering across separate modules with links
- Module links maintain isolation between modules

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Architecture Violations | CRITICAL | `arch-` |
| 2 | Type Safety | CRITICAL | `type-` |
| 3 | Business Logic Placement | HIGH | `logic-` |
| 4 | Import & Code Organization | HIGH | `import-` |
| 5 | Data Access Patterns | MEDIUM (includes CRITICAL price rule) | `data-` |
| 6 | File Organization | MEDIUM | `file-` |

## Quick Reference

### 1. Architecture Violations (CRITICAL)

- `arch-workflow-required` - Use workflows for ALL mutations, never call module services from routes
- `arch-layer-bypass` - Never bypass layers (route → service without workflow)
- `arch-http-methods` - Use only GET, POST, DELETE (never PUT/PATCH)
- `arch-module-isolation` - Use module links, not direct cross-module service calls
- `arch-query-config-fields` - Don't set explicit `fields` when using `req.queryConfig`

### 2. Type Safety (CRITICAL)

- `type-request-schema` - Pass Zod inferred type to `MedusaRequest<T>` when using `req.validatedBody`
- `type-authenticated-request` - Use `AuthenticatedMedusaRequest` for protected routes (not `MedusaRequest`)
- `type-export-schema` - Export both Zod schema AND inferred type from middlewares
- `type-linkable-auto` - Never add `.linkable()` to data models (automatically added)
- `type-module-name-camelcase` - Module names MUST be camelCase, never use dashes (causes runtime errors)

### 3. Business Logic Placement (HIGH)

- `logic-workflow-validation` - Put business validation in workflow steps, not API routes
- `logic-ownership-checks` - Validate ownership/permissions in workflows, not routes
- `logic-module-service` - Keep modules simple (CRUD only), put logic in workflows

### 4. Import & Code Organization (HIGH)

- `import-top-level` - Import workflows/modules at file top, never use `await import()` in route body
- `import-static-only` - Use static imports for all dependencies
- `import-no-dynamic-routes` - Dynamic imports add overhead and break type checking

### 5. Data Access Patterns (MEDIUM)

- `data-price-format` - **CRITICAL**: Prices are stored as-is in Medusa (49.99 stored as 49.99, NOT in cents). Never multiply by 100 when saving or divide by 100 when displaying
- `data-query-method` - Use `query.graph()` for cross-module queries, `query.index()` for filtered queries across linked modules
- `data-soft-delete` - Soft-deleted records are excluded by default, use `withDeleted: true` if needed

### 6. File Organization (MEDIUM)

- `file-api-structure` - API routes: `api/[scope]/[feature]/route.ts`, middlewares: `api/[scope]/[feature]/middlewares.ts`
- `file-workflow-structure` - Workflows: `src/workflows/[name].ts`, steps: `src/workflows/steps/[step-name].ts`
- `file-module-structure` - Modules: `src/modules/[name]/models/`, `src/modules/[name]/service.ts`, `src/modules/[name]/index.ts`

## Implementation Checklist Template

When implementing Medusa features, follow this order:

1. **Plan the architecture** - Identify modules, workflows, and routes needed
2. **Create data models** - Define entities in module models
3. **Create module service** - Extend MedusaService with CRUD operations
4. **Register module** - Add to medusa-config.ts BEFORE using
5. **Generate migrations** - `npx medusa db:generate [module-name]`
6. **Run migrations** - `npx medusa db:migrate`
7. **Create workflows** - Define business logic with steps and compensation
8. **Create API routes** - Build HTTP endpoints with validation
9. **Register middlewares** - Add validation and authentication
10. **Build and test** - `npx medusa build` then test endpoints

## Common Pitfalls

- ❌ Forgetting to register module in medusa-config.ts before using it
- ❌ Skipping migration generation/running (database tables won't exist)
- ❌ Using PUT/PATCH instead of POST for mutations
- ❌ Putting business logic in API routes instead of workflows
- ❌ Using dashes in module names (must be camelCase)
- ❌ Calling module services directly from routes (use workflows)
- ❌ Multiplying/dividing prices by 100 (Medusa stores prices as-is)
- ❌ Forgetting to export Zod schema types from middlewares
- ❌ Using dynamic imports for workflows/modules
- ❌ Not running `npx medusa build` after making changes
