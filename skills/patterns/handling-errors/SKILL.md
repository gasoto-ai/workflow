---
name: handling-errors
type: pattern
description: Enforces error handling patterns including error bubbling, progressive degradation, and user-friendly messages. This is reference material — consult during implementation for error handling conventions.
---

# Handling Errors

Patterns for proper error handling — errors bubble up, UI degrades gracefully.

## Core Principle — Let Errors Bubble

```typescript
// Bad: Swallowing errors
try {
    const result = await submitForm(data);
    return result;
} catch (error) {
    console.log("Error:", error); // Swallowed!
    return null;
}

// Good: Let it bubble, handle at the boundary
const result = await submitForm(data); // Throws naturally
```

Catch errors at **boundaries** (API handlers, form submit handlers, error boundaries), not in every function.

## Don't Catch and Return

```typescript
// Bad: Catch, mutate, return
const fetchProducts = async (country: string) => {
    try {
        const products = await getCatalog(country);
        return { success: true, data: products };
    } catch (error) {
        return { success: false, error: error.message };
    }
};

// Good: Throw, let caller handle
const fetchProducts = async (country: string) => {
    return getCatalog(country); // Caller catches if needed
};
```

## React Query Error Handling

React Query manages error state — don't catch errors inside `queryFn`.

```tsx
// BAD: Catching in queryFn (swallows the error)
const { data } = useQuery({
    queryKey: ["catalog", country],
    queryFn: async () => {
        try {
            return await getCatalog(country);
        } catch {
            return []; // Error is hidden from React Query!
        }
    },
});

// GOOD: Let queryFn throw, use error state
const { data, error, isLoading } = useQuery({
    queryKey: ["catalog", country],
    queryFn: () => getCatalog(country),
});

if (error) return <ErrorMessage error={error} />;
```

### Mutation Error Handling

For mutations, handle errors in the form's `onSubmit` with `setError`:

```tsx
const { mutateAsync } = useSubmitFormMutation();

const onSubmit = async (data: FormData) => {
    try {
        await mutateAsync(data);
        router.push("/thank-you");
    } catch (error) {
        if (error instanceof ApiError && error.statusCode === 422) {
            // Field-specific errors from API
            setError("email", { message: "This email is already registered." });
        } else {
            // Root-level error for unexpected failures
            setError("root", {
                message: getUserFriendlyMessage(error),
            });
        }
    }
};
```

## Form Error Handling

```tsx
const onSubmit = async (data: FormData) => {
    try {
        await submitForm(data);
        router.push("/thank-you");
    } catch (error) {
        if (error instanceof ValidationError) {
            setError("email", { message: error.fieldMessage });
        } else {
            setError("root", {
                message: "Something went wrong. Please try again.",
            });
        }
    }
};
```

## Progressive Degradation — Don't Unmount on Error

```tsx
// Bad: Error replaces entire UI
if (error) return <ErrorPage />;

// Good: Show error alongside working UI
return (
    <div>
        {error && (
            <div role="alert" data-testid="enrollment-error-message">
                {getUserFriendlyMessage(error)}
            </div>
        )}
        <ContactForm /> {/* Still usable */}
    </div>
);
```

## User-Friendly Error Messages

```typescript
const getUserFriendlyMessage = (error: unknown): string => {
    if (error instanceof ApiError) {
        switch (error.statusCode) {
            case 409:
                return "This email is already registered. Try logging in instead.";
            case 422:
                return "Some of your information couldn't be verified. Please check and try again.";
            case 503:
                return "Our service is temporarily unavailable. Please try again in a few minutes.";
            default:
                return "Something went wrong. Please try again.";
        }
    }

    if (error instanceof PaymentError) {
        return error.userMessage ?? "Payment could not be processed. Please try a different method.";
    }

    return "An unexpected error occurred. Please try again.";
};

// Never expose to users:
// - Stack traces
// - API error codes or internal messages
// - Database details
// - Third-party service names
```

## Error Boundaries (React)

```tsx
// app/error.tsx — catches unhandled errors per route
"use client";

const ErrorBoundary = ({
    error,
    reset,
}: {
    error: Error & { digest?: string };
    reset: () => void;
}) => {
    return (
        <div data-testid="error-boundary">
            <h2>Something went wrong</h2>
            <p>We encountered an unexpected error. Please try again.</p>
            <button onClick={reset} data-testid="error-retry-button">
                Try again
            </button>
        </div>
    );
};

export default ErrorBoundary;
```

## API Route Error Handling

```typescript
// app/api/catalog/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
    const country = request.nextUrl.searchParams.get("country");

    if (!country) {
        return NextResponse.json(
            { error: "Country parameter is required" },
            { status: 400 }
        );
    }

    try {
        const catalog = await getCatalog(country);
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

## Checklist

- [ ] Errors bubble up — only caught at boundaries (form handlers, API routes, error boundaries)
- [ ] No `catch` blocks that swallow errors (log + return null)
- [ ] User-facing messages are friendly — no technical details exposed
- [ ] Form errors use `setError` (field-specific or root)
- [ ] UI degrades gracefully — error shown alongside working UI, not replacing it
- [ ] Error boundaries (`error.tsx`) exist for route-level failures
- [ ] API routes return consistent error shape: `{ error: string }`
- [ ] Errors from external services are mapped to user-friendly messages
- [ ] React Query `queryFn` does NOT catch errors — uses `error` state instead
- [ ] Mutation errors caught in `onSubmit` and displayed via `setError`
