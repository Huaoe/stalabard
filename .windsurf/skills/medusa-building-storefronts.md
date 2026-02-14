---
name: medusa-building-storefronts
description: Load when building storefronts that call Medusa backend APIs. Covers SDK integration, React Query, and best practices for storefront development.
---

# Building Storefronts with Medusa

Guide for building storefronts that integrate with Medusa backend APIs.

## Medusa SDK Setup

Initialize the Medusa SDK in your storefront:

```typescript
// lib/medusa.ts
import Medusa from "@medusajs/js-sdk"

export const medusa = new Medusa({
  baseUrl: process.env.NEXT_PUBLIC_MEDUSA_URL || "http://localhost:9000",
  maxRetries: 3,
})
```

## Using Medusa SDK with React Query

Combine Medusa SDK with React Query for data fetching:

```typescript
// hooks/useProducts.ts
import { useQuery } from "@tanstack/react-query"
import { medusa } from "@/lib/medusa"

export const useProducts = () => {
  return useQuery({
    queryKey: ["products"],
    queryFn: async () => {
      const { products } = await medusa.store.product.list()
      return products
    },
  })
}
```

## Calling Custom Store API Routes

Call your custom store API routes from the storefront:

```typescript
// hooks/useCustomFeature.ts
import { useQuery, useMutation } from "@tanstack/react-query"
import { medusa } from "@/lib/medusa"

export const useCustomFeature = () => {
  return useQuery({
    queryKey: ["custom-feature"],
    queryFn: async () => {
      const response = await fetch(
        `${process.env.NEXT_PUBLIC_MEDUSA_URL}/store/custom-feature`,
        {
          headers: {
            "x-publishable-api-key": process.env.NEXT_PUBLIC_PUBLISHABLE_KEY!,
          },
        }
      )
      return response.json()
    },
  })
}

export const useCreateCustomFeature = () => {
  return useMutation({
    mutationFn: async (data: any) => {
      const response = await fetch(
        `${process.env.NEXT_PUBLIC_MEDUSA_URL}/store/custom-feature`,
        {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "x-publishable-api-key": process.env.NEXT_PUBLIC_PUBLISHABLE_KEY!,
          },
          body: JSON.stringify(data),
        }
      )
      return response.json()
    },
  })
}
```

## Authentication in Storefronts

Use the Medusa SDK's built-in authentication:

```typescript
// hooks/useAuth.ts
import { useQuery } from "@tanstack/react-query"
import { medusa } from "@/lib/medusa"

export const useMe = () => {
  return useQuery({
    queryKey: ["me"],
    queryFn: async () => {
      const { customer } = await medusa.store.customer.retrieve()
      return customer
    },
  })
}

export const useLogin = () => {
  return useMutation({
    mutationFn: async (credentials: { email: string; password: string }) => {
      const { customer } = await medusa.store.customer.login(credentials)
      return customer
    },
  })
}
```

## Environment Variables

Set up required environment variables:

```bash
# .env.local
NEXT_PUBLIC_MEDUSA_URL=http://localhost:9000
NEXT_PUBLIC_PUBLISHABLE_KEY=your_publishable_key
```

## Error Handling

Handle API errors gracefully:

```typescript
export const useCustomFeature = () => {
  return useQuery({
    queryKey: ["custom-feature"],
    queryFn: async () => {
      try {
        const response = await fetch(
          `${process.env.NEXT_PUBLIC_MEDUSA_URL}/store/custom-feature`
        )
        if (!response.ok) {
          throw new Error(`API error: ${response.status}`)
        }
        return response.json()
      } catch (error) {
        console.error("Failed to fetch custom feature:", error)
        throw error
      }
    },
  })
}
```

## Best Practices

- **Use React Query**: For caching, synchronization, and state management
- **Separate hooks**: Create custom hooks for each API endpoint
- **Handle loading states**: Show loading indicators while fetching
- **Handle errors**: Display user-friendly error messages
- **Use environment variables**: Don't hardcode API URLs
- **Validate data**: Validate API responses before using
- **Cache strategically**: Use React Query's cache configuration
- **Optimize queries**: Use pagination and filtering

## Implementation Checklist

- [ ] Set up Medusa SDK
- [ ] Configure React Query
- [ ] Create custom hooks for API endpoints
- [ ] Handle authentication
- [ ] Set up environment variables
- [ ] Implement error handling
- [ ] Test API integration
- [ ] Optimize data fetching
