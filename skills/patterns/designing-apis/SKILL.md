---
name: designing-apis
type: pattern
description: Designs REST API route handlers with Zod validation and consistent error responses for Next.js App Router. This is reference material — consult during implementation for API conventions.
---

# API Design Patterns

REST API design for Next.js 15 App Router with Zod validation and consistent error handling.

## Route Handler Pattern

```typescript
// app/api/catalog/route.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";

const querySchema = z.object({
    country: z.string().length(2),
    language: z.string().length(2).optional().default("en"),
});

export async function GET(request: NextRequest) {
    // 1. Validate input
    const params = Object.fromEntries(request.nextUrl.searchParams);
    const parsed = querySchema.safeParse(params);

    if (!parsed.success) {
        return NextResponse.json(
            { error: "Invalid parameters", details: parsed.error.flatten().fieldErrors },
            { status: 400 }
        );
    }

    // 2. Handle request
    try {
        const catalog = await getProducts(parsed.data.country, parsed.data.language);
        return NextResponse.json(catalog);
    } catch (error) {
        console.error("Catalog fetch failed:", error);
        return NextResponse.json(
            { error: "Failed to fetch catalog" },
            { status: 500 }
        );
    }
}
```

## Request Validation with Zod

Use `safeParse` in route handlers — you control the error response. Service functions use `.parse()` instead (let errors bubble). The rule is layer-dependent: route handlers = `safeParse`, service functions = `parse`.

```typescript
// POST body validation
const enrollmentSchema = z.object({
    firstName: z.string().min(1),
    lastName: z.string().min(1),
    email: z.string().email(),
    country: z.string().length(2),
    products: z.array(z.string()).min(1),
});

export async function POST(request: NextRequest) {
    const body = await request.json();
    const parsed = enrollmentSchema.safeParse(body);

    if (!parsed.success) {
        return NextResponse.json(
            { error: "Validation failed", details: parsed.error.flatten().fieldErrors },
            { status: 400 }
        );
    }

    // parsed.data is fully typed
    const result = await processEnrollment(parsed.data);
    return NextResponse.json(result, { status: 201 });
}
```

## Error Response Format

Consistent shape across all API routes:

```typescript
// Success — return data directly (no wrapper)
NextResponse.json(catalog)
NextResponse.json(result, { status: 201 })

// Client error (400, 404, 422)
{ error: "Human-readable message", details?: { field: ["message"] } }

// Server error (500)
{ error: "Failed to process request" }
// Never expose: stack traces, SQL errors, internal service names
```

## Server Actions

For form submissions that don't need a REST endpoint:

```typescript
// app/actions/submit-form.ts
"use server";

import { z } from "zod";

const schema = z.object({
    firstName: z.string().min(1),
    email: z.string().email(),
});

export async function submitForm(formData: FormData) {
    const parsed = schema.safeParse(Object.fromEntries(formData));

    if (!parsed.success) {
        return { error: parsed.error.flatten().fieldErrors };
    }

    const result = await apiClient.createOrder(parsed.data);
    return { orderId: result.id };
}
```

## Dynamic Route Parameters

```typescript
// app/api/catalog/[country]/route.ts
export async function GET(
    request: NextRequest,
    { params }: { params: Promise<{ country: string }> }
) {
    const { country } = await params; // Next.js 15: params is async
    const catalog = await getProducts(filters);
    return NextResponse.json(catalog);
}
```

## Health Check Pattern

```typescript
// app/api/health/route.ts
export async function GET() {
    return NextResponse.json({
        status: "healthy",
        timestamp: new Date().toISOString(),
    });
}
```

## Service Function Integration

Route handlers can call API service functions directly. Service functions handle Zod validation of the API response; the route handler validates the incoming request.

```typescript
// app/api/catalog/[country]/route.ts
import { getProducts } from "services/api/getProducts";

export async function GET(
    request: NextRequest,
    { params }: { params: Promise<{ country: string }> }
) {
    const { country } = await params;

    try {
        // Service function validates API response with Zod
        const catalog = await getProducts(filters);
        return NextResponse.json(catalog);
    } catch (error) {
        console.error("Catalog fetch failed:", error);
        return NextResponse.json(
            { error: "Failed to fetch catalog" },
            { status: 500 }
        );
    }
}
```

See `data-fetching` skill for service function patterns.

## Checklist

- [ ] Route handler input validated with Zod `safeParse` (service functions use `parse`)
- [ ] Consistent error response shape: `{ error, details? }`
- [ ] No internal details in error responses
- [ ] `params` awaited (Next.js 15 async)
- [ ] Server actions for form submissions, route handlers for REST APIs
- [ ] Schemas defined in the route file or colocated nearby
- [ ] Types inferred from schemas (`z.infer<typeof schema>`)
- [ ] Route handlers call service functions (not raw fetch) for external API data
