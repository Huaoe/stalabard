---
description: Create a new custom Medusa module with data models, service, and migrations
---

# Create Custom Medusa Module

This workflow guides you through creating a complete custom module in Medusa.

## Prerequisites

- Medusa project initialized
- Understanding of the module structure (see medusa-reference-custom-modules skill)

## Steps

1. **Load the medusa-reference-custom-modules skill**
   - This provides detailed guidance on module structure and best practices

2. **Create the data model**
   - Create `src/modules/[module-name]/models/[entity-name].ts`
   - Define all fields using Medusa's model utilities
   - Remember: `created_at`, `updated_at`, `deleted_at` are auto-added

3. **Create the service**
   - Create `src/modules/[module-name]/service.ts`
   - Extend `MedusaService` with your models
   - Auto-generated CRUD methods will be available

4. **Export module definition**
   - Create `src/modules/[module-name]/index.ts`
   - Use `Module()` to define the module
   - **CRITICAL:** Use camelCase for module names (never dashes)

5. **Register in medusa-config.ts**
   - Add module to the `modules` array in configuration
   - **CRITICAL:** Do this BEFORE generating migrations

6. **Generate migrations**
   - Run: `npx medusa db:generate [module-name]`
   - This creates migration files for your data models

7. **Run migrations**
   - Run: `npx medusa db:migrate`
   - This applies changes to the database

8. **Build and validate**
   - Run: `npx medusa build`
   - Catches type errors and configuration issues

## Validation Checklist

- [ ] Module name is camelCase (no dashes)
- [ ] Data models defined with correct fields
- [ ] Service extends MedusaService
- [ ] Module registered in medusa-config.ts
- [ ] Migrations generated and run successfully
- [ ] Build completes without errors
- [ ] Can resolve module from container: `container.resolve("[module-name]")`

## Common Issues

**Module not found at runtime**
- Check module name is camelCase in index.ts
- Verify module is registered in medusa-config.ts
- Ensure migrations were run

**Database tables don't exist**
- Run: `npx medusa db:generate [module-name]`
- Run: `npx medusa db:migrate`
- Never skip these steps

**Type errors after creating module**
- Run: `npx medusa build`
- Check all imports are correct
- Verify service extends MedusaService properly
