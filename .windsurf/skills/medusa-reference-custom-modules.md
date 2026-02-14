---
name: medusa-reference-custom-modules
description: Reference guide for creating custom modules in Medusa. Load when implementing custom modules, data models, and module services.
---

# Custom Modules Reference

## When to Create a Custom Module

- **New domain concepts**: Brands, wishlists, reviews, loyalty points
- **Third-party integrations**: ERPs, CMSs, custom services
- **Isolated business logic**: Features that don't fit existing commerce modules

## Module Structure

```
src/modules/blog/
├── models/
│   └── post.ts          # Data model definitions
├── service.ts           # Main service class
└── index.ts             # Module definition export
```

## Step 1: Create the Data Model

```typescript
// src/modules/blog/models/post.ts
import { model } from "@medusajs/framework/utils"

const Post = model.define("post", {
  id: model.id().primaryKey(),
  title: model.text(),
  content: model.text().nullable(),
  published: model.boolean().default(false),
})

// note models automatically get created_at, updated_at and deleted_at added - don't add these explicitly

export default Post
```

**Note:** Models automatically get `created_at`, `updated_at`, and `deleted_at` fields - don't add these explicitly.

## Step 2: Create the Service

```typescript
// src/modules/blog/service.ts
import { MedusaService } from "@medusajs/framework/utils"
import Post from "./models/post"

class BlogModuleService extends MedusaService({
  Post,
}) {}

export default BlogModuleService
```

The service extends `MedusaService` which auto-generates CRUD methods for each data model.

## Step 3: Export Module Definition

```typescript
// src/modules/blog/index.ts
import BlogModuleService from "./service"
import { Module } from "@medusajs/framework/utils"

export const BLOG_MODULE = "blog"

export default Module(BLOG_MODULE, {
  service: BlogModuleService,
})
```

**⚠️ CRITICAL - Module Name Format:**
- Module names MUST be in camelCase
- **NEVER use dashes (kebab-case)** in module names
- ✅ CORRECT: `"blog"`, `"productReview"`, `"orderTracking"`
- ❌ WRONG: `"product-review"`, `"order-tracking"` (will cause runtime errors)

**Example of common mistake:**

```typescript
// ❌ WRONG - dashes will break the module
export const PRODUCT_REVIEW_MODULE = "product-review" // Don't do this!
export default Module("product-review", { service: ProductReviewService })

// ✅ CORRECT - use camelCase
export const PRODUCT_REVIEW_MODULE = "productReview"
export default Module("productReview", { service: ProductReviewService })
```

**Why this matters:** Medusa's internal module resolution uses property access syntax (e.g., `container.resolve("productReview")`), and dashes would break this.

## Step 4: Register in Configuration

**IMPORTANT**: You MUST register the module in the configurations BEFORE using it anywhere or generating migrations.

```typescript
// medusa-config.ts
module.exports = defineConfig({
  // ...
  modules: [{ resolve: "./src/modules/blog" }],
})
```

## Steps 5-6: Generate and Run Migrations

**⚠️ CRITICAL - DO NOT SKIP**: After creating a module and registering it in medusa-config.ts, you MUST run TWO SEPARATE commands. Without this step, the module's database tables won't exist and you will get runtime errors.

```bash
# Step 5: Generate migrations (creates migration files)
# Command format: npx medusa db:generate <module-name>
npx medusa db:generate blog

# Step 6: Run migrations (applies changes to database)
npx medusa db:migrate
```

## Auto-Generated CRUD Methods

When you extend `MedusaService`, it automatically generates these methods for each model:

```typescript
// For a Post model, these methods are auto-generated:

// Create
await blogService.createPosts({ title: "My Post", content: "..." })

// Read
await blogService.retrievePost(postId)
await blogService.listPosts()
await blogService.listAndCountPosts()

// Update
await blogService.updatePosts(postId, { title: "Updated" })

// Delete
await blogService.deletePosts(postId)
```

## Resolving Services from Container

In workflows and API routes, resolve the module service from the container:

```typescript
// In a workflow step
const blogService = container.resolve("blog")
const posts = await blogService.listPosts()

// In an API route
const blogService = req.scope.resolve("blog")
const post = await blogService.retrievePost(postId)
```

## Loaders

Loaders allow you to automatically load related data when retrieving models:

```typescript
// src/modules/blog/models/post.ts
import { model } from "@medusajs/framework/utils"

const Post = model.define("post", {
  id: model.id().primaryKey(),
  title: model.text(),
  author_id: model.text(),
  // ... other fields
})

export default Post
```

Use loaders in queries to automatically fetch related data:

```typescript
const post = await blogService.retrievePost(postId, {
  relations: ["author"],
})
```

## Data Model Field Types

Common field types available in Medusa models:

```typescript
import { model } from "@medusajs/framework/utils"

const MyModel = model.define("myModel", {
  id: model.id().primaryKey(),
  
  // Text fields
  name: model.text(),
  description: model.text().nullable(),
  
  // Number fields
  quantity: model.integer(),
  price: model.bigNumber(),
  
  // Boolean
  active: model.boolean().default(true),
  
  // Date
  published_at: model.dateTime().nullable(),
  
  // Relationships
  category_id: model.text(),
})

export default MyModel
```

## Implementation Checklist

- [ ] Create data model in `src/modules/[name]/models/`
- [ ] Create service extending MedusaService
- [ ] Export module definition in `index.ts`
- [ ] **CRITICAL:** Register module in `medusa-config.ts`
- [ ] **CRITICAL:** Generate migrations: `npx medusa db:generate [module-name]`
- [ ] **CRITICAL:** Run migrations: `npx medusa db:migrate`
- [ ] Use module service in API routes/workflows
- [ ] **CRITICAL:** Run build to validate: `npx medusa build`
