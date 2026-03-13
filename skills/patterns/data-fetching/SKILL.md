---
name: data-fetching
type: pattern
description: Enforces React Query patterns for data fetching, service functions with Zod validation, and server/client data flow. This is reference material — consult during implementation for data fetching conventions.
---

# Data Fetching

Architecture: Component -> React Query hook -> service function. Server components call service functions directly.

## Service Function Pattern

Each service function lives in its own file under `src/services/` (e.g., `src/services/api/`). Validates response with Zod at the boundary. No try/catch — let errors bubble.

```typescript
// src/services/api/getUser.ts
import { z } from "zod";
import { apiFetch } from "services/api/apiFetch";

const UserResponseSchema = z.object({
    id: z.string(),
    email: z.string().email(),
    firstName: z.string(),
    lastName: z.string(),
    country: z.string().length(2),
});

export type UserResponse = z.infer<typeof UserResponseSchema>;

export const getUser = async (token: string) => {
    const data = await apiFetch("/me", {
        headers: { Authorization: `Bearer ${token}` },
    });
    return UserResponseSchema.parse(data);
};
```

Key rules:
- One function per file, named for the action (`getUser.ts`, `getProducts.ts`, `submitOrder.ts`)
- Zod schema validates the response — this is the boundary
- Type is inferred from schema (`z.infer<typeof Schema>`)
- No try/catch — caller handles errors
- No `{ success: true, data }` wrappers — return data directly or throw

## Typed HTTP Client

`apiFetch` is a typed fetch wrapper. Use this for all service functions.

```typescript
// src/services/api/apiFetch.ts
type ApiFetchOptions = {
    method?: "GET" | "POST" | "PUT" | "DELETE";
    headers?: Record<string, string>;
    body?: unknown;
};

export const apiFetch = async (path: string, options: ApiFetchOptions = {}) => {
    const baseUrl = process.env.NEXT_PUBLIC_API_URL;
    const response = await fetch(`${baseUrl}${path}`, {
        method: options.method ?? "GET",
        headers: {
            "Content-Type": "application/json",
            ...options.headers,
        },
        body: options.body ? JSON.stringify(options.body) : undefined,
    });

    if (!response.ok) {
        throw new ApiError(response.status, await response.text());
    }

    return response.json();
};
```

## React Query Hook Pattern

Query hooks live in `src/queries/` and wrap service functions with `useQuery`.

```tsx
// src/queries/useUserQuery.ts
import { useQuery } from "@tanstack/react-query";
import { getUser } from "services/api/getUser";

export const useUserQuery = (token: string | null) =>
    useQuery({
        queryKey: ["me", token],
        queryFn: () => getUser(token!),
        enabled: !!token,
    });
```

```tsx
// src/queries/useProductsQuery.ts
import { useQuery } from "@tanstack/react-query";
import { getProducts } from "services/api/getProducts";

export const useProductsQuery = (country: string) =>
    useQuery({
        queryKey: ["catalog", country],
        queryFn: () => getProducts(country),
        staleTime: 5 * 60 * 1000, // Catalog changes infrequently
    });
```

Naming: `use{Entity}Query.ts` for queries, `use{Action}Mutation.ts` for mutations.

## Query Key Conventions

Hierarchical keys for targeted invalidation:

```typescript
// Entity-based
["me"]                          // Current user
["me", token]                   // Current user by token
["catalog", country]            // Catalog for country
["catalog", country, language]  // Catalog for country+language
["order", orderId]              // Specific order

// Invalidate all catalog queries
queryClient.invalidateQueries({ queryKey: ["catalog"] });

// Invalidate only US catalog
queryClient.invalidateQueries({ queryKey: ["catalog", "US"] });
```

## Mutation Pattern

```tsx
// src/queries/useSubmitOrderMutation.ts
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { submitOrder } from "services/api/submitOrder";

export const useSubmitOrderMutation = () => {
    const queryClient = useQueryClient();

    return useMutation({
        mutationFn: submitOrder,
        onSuccess: () => {
            queryClient.invalidateQueries({ queryKey: ["me"] });
        },
    });
};
```

Using in a form:

```tsx
const { mutateAsync, isPending } = useSubmitOrderMutation();

const onSubmit = async (data: OrderData) => {
    try {
        await mutateAsync(data);
        router.push("/thank-you");
    } catch (error) {
        setError("root", { message: getUserFriendlyMessage(error) });
    }
};
```

## Server Component Data Fetching

Server components call service functions directly — no React Query needed.

```tsx
// app/home/page.tsx (server component)
import { getProducts } from "services/api/getProducts";
import { cookies } from "next/headers";

export default async function HomePage() {
    const cookieStore = await cookies();
    const country = cookieStore.get("country")?.value ?? "US";
    const catalog = await getProducts(country);

    return <ProductSelection products={catalog.items} />;
}
```

## Prefetching & Hydration

For pages that render server-side but need React Query on the client:

```tsx
// app/register/page.tsx
import { dehydrate, HydrationBoundary, QueryClient } from "@tanstack/react-query";
import { getProducts } from "services/api/getProducts";

export default async function RegisterPage() {
    const queryClient = new QueryClient();
    const country = "US"; // from cookies

    await queryClient.prefetchQuery({
        queryKey: ["catalog", country],
        queryFn: () => getProducts(country),
    });

    return (
        <HydrationBoundary state={dehydrate(queryClient)}>
            <RegisterFlow />
        </HydrationBoundary>
    );
}
```

Client component receives hydrated cache — no loading flash:

```tsx
// RegisterFlow.tsx (client component)
"use client";

const RegisterFlow = () => {
    // Data is already in cache from server prefetch — instant
    const { data: catalog } = useProductsQuery("US");
    return <ProductSelection products={catalog.items} />;
};
```

## Error Handling in Queries

Don't catch errors in `queryFn`. React Query manages error state.

```tsx
// Component uses error state from the hook
const { data, error, isLoading } = useProductsQuery(country);

if (isLoading) return <Skeleton />;
if (error) return <ErrorMessage error={error} />;
return <ProductList products={data.items} />;
```

For mutations, handle errors in `onSubmit` with `setError` — see `handling-errors` skill.

## Anti-Patterns

```tsx
// BAD: useEffect + useState for data fetching
const [products, setProducts] = useState([]);
const [isLoading, setIsLoading] = useState(false);
useEffect(() => {
    setIsLoading(true);
    fetchProducts().then(setProducts).finally(() => setIsLoading(false));
}, []);

// GOOD: React Query
const { data: products, isLoading } = useProductsQuery();

// BAD: Duplicating query data to context
const { data } = useProductsQuery(country);
useEffect(() => {
    setProductContext(data); // Don't copy to context!
}, [data]);

// GOOD: Use query data directly
const { data: catalog } = useProductsQuery(country);
return <ProductList products={catalog.items} />;

// BAD: Catching errors in queryFn
const { data } = useQuery({
    queryKey: ["catalog"],
    queryFn: async () => {
        try {
            return await getProducts(country);
        } catch {
            return []; // Swallowed error!
        }
    },
});

// GOOD: Let queryFn throw, use error state
const { data, error } = useQuery({
    queryKey: ["catalog", country],
    queryFn: () => getProducts(country),
});
```

## Checklist

- [ ] Service function in its own file under `src/services/api/`
- [ ] Response validated with Zod schema at boundary
- [ ] Type inferred from schema (`z.infer<typeof Schema>`)
- [ ] No try/catch in service functions — errors bubble
- [ ] Query hook in `src/queries/` with `use{Entity}Query` naming
- [ ] Query keys are hierarchical: `[entity, ...identifiers]`
- [ ] Mutations invalidate related queries on success
- [ ] Server components call service functions directly (no React Query)
- [ ] Prefetch + HydrationBoundary for server-to-client handoff
- [ ] No `useEffect` + `useState` for data fetching
- [ ] Query data not duplicated to context or global state
- [ ] Error state from `useQuery` displayed in UI (not swallowed)
