# Medusa Skills for Windsurf

This directory contains comprehensive skills and workflows for Medusa development in Windsurf. These are adapted from the official [medusa-agent-skills](https://github.com/medusajs/medusa-agent-skills) repository.

## Overview

The Medusa skills provide guidance on best practices, architectural patterns, and implementation details for building Medusa applications. They cover:

- **Backend development** - Modules, workflows, API routes
- **Data modeling** - Custom entities and relationships
- **Authentication & Authorization** - Protected routes and user management
- **Storefronts** - SDK integration and client-side development
- **Debugging** - Common issues and solutions

## Skills Structure

### Core Skills

**`medusa-building-with-medusa.md`**
- Main skill for all backend development
- Architecture patterns and critical rules
- Load this for ANY backend work

### Reference Skills

**`medusa-reference-custom-modules.md`**
- Creating custom modules and data models
- Module structure and registration
- Auto-generated CRUD methods

**`medusa-reference-workflows.md`**
- Creating workflows for mutations
- Workflow composition rules
- Step functions and compensation

**`medusa-reference-api-routes.md`**
- Building HTTP endpoints
- Request validation with Zod
- Error handling and responses

**`medusa-reference-authentication.md`**
- Protected routes and authentication
- Customer vs user authentication
- Authorization patterns

**`medusa-reference-querying-data.md`**
- Querying across modules
- Filtering and pagination
- Cross-module queries with query.graph()

**`medusa-reference-module-links.md`**
- Defining relationships between modules
- Module isolation patterns
- Linked data queries

**`medusa-building-storefronts.md`**
- Storefront integration
- Medusa SDK setup
- React Query patterns

### Workflow Guides

**`medusa-setup-project.md`**
- Initial project setup
- Configuration and environment
- Database initialization

**`medusa-create-custom-module.md`**
- Step-by-step module creation
- Data models and services
- Migration generation and running

**`medusa-create-workflow.md`**
- Workflow implementation guide
- Composition rules and patterns
- Step functions and compensation

**`medusa-create-api-route.md`**
- API route creation
- Validation and middleware
- Authentication integration

**`medusa-debug-issues.md`**
- Troubleshooting common problems
- Debugging techniques
- Error resolution

## How to Use

### For Backend Development

1. **Load `medusa-building-with-medusa`** for architectural guidance
2. **Load relevant reference skill** based on what you're implementing:
   - Creating module? → Load `medusa-reference-custom-modules`
   - Creating workflow? → Load `medusa-reference-workflows`
   - Creating API route? → Load `medusa-reference-api-routes`
   - Adding authentication? → Load `medusa-reference-authentication`
   - Querying data? → Load `medusa-reference-querying-data`

3. **Follow the implementation checklist** in the skill
4. **Use workflow guides** for step-by-step instructions

### For Storefronts

1. **Load `medusa-building-storefronts`** for SDK integration
2. **Follow patterns** for React Query and data fetching
3. **Reference authentication** skill for protected endpoints

### For Troubleshooting

1. **Load `medusa-debug-issues`** when encountering problems
2. **Follow diagnostic steps** to identify root cause
3. **Apply suggested fixes** from the skill

## MCP Server Integration

The `.windsurf/mcp.json` file configures the Medusa documentation MCP server:

```json
{
  "mcpServers": {
    "MedusaDocs": {
      "type": "http",
      "url": "https://docs.medusajs.com/mcp"
    }
  }
}
```

This provides access to official Medusa documentation for reference.

## Critical Rules Summary

### Architecture (CRITICAL)

- ✅ Use workflows for ALL mutations
- ✅ Follow: Module → Workflow → API Route → Frontend
- ✅ Use only GET, POST, DELETE (never PUT/PATCH)
- ✅ Use module links for cross-module relationships

### Type Safety (CRITICAL)

- ✅ Pass Zod schema type to `MedusaRequest<T>`
- ✅ Use `AuthenticatedMedusaRequest` for protected routes
- ✅ Export both schema AND type from middlewares
- ✅ Module names MUST be camelCase (never dashes)

### Business Logic (HIGH)

- ✅ Put validation in workflows, not routes
- ✅ Keep modules simple (CRUD only)
- ✅ Put business logic in workflow steps

### Data (MEDIUM)

- ✅ Prices stored as-is (49.99 = 49.99, not 4999)
- ✅ Use `query.graph()` for cross-module queries
- ✅ Soft-deleted records excluded by default

## Common Pitfalls

❌ **Don't:**
- Forget to register module in medusa-config.ts
- Skip migration generation/running
- Use PUT/PATCH for mutations
- Put business logic in API routes
- Use dashes in module names
- Call module services directly from routes
- Multiply/divide prices by 100
- Use dynamic imports for workflows

✅ **Do:**
- Register modules BEFORE using them
- Always run migrations after creating modules
- Use workflows for all mutations
- Keep modules focused on CRUD
- Use camelCase for module names
- Use workflows for business logic
- Store prices as-is
- Import workflows at top level

## Implementation Checklist Template

For any Medusa feature:

1. **Plan** - Identify modules, workflows, routes needed
2. **Create data models** - Define entities
3. **Create service** - Extend MedusaService
4. **Register module** - Add to medusa-config.ts
5. **Generate migrations** - `npx medusa db:generate [name]`
6. **Run migrations** - `npx medusa db:migrate`
7. **Create workflows** - Define business logic
8. **Create API routes** - Build HTTP endpoints
9. **Register middlewares** - Add validation/auth
10. **Build & test** - `npx medusa build` then test

## Resources

- **Official Docs**: https://docs.medusajs.com
- **GitHub**: https://github.com/medusajs/medusa
- **Agent Skills**: https://github.com/medusajs/medusa-agent-skills
- **Community**: https://discord.gg/medusajs

## Version

- **Medusa Skills Version**: 1.0.8
- **Last Updated**: February 2026
- **Adapted for Windsurf**: Yes

## Notes

These skills are adapted from the official Medusa agent skills repository and optimized for Windsurf integration. They maintain the same architectural patterns and best practices while being formatted for Windsurf's skill system.

All skills include:
- Clear prerequisites
- Step-by-step guidance
- Code examples
- Implementation checklists
- Common pitfalls and solutions
- Validation steps
