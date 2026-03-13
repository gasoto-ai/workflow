---
name: writing-typescript
type: pattern
description: Enforces TypeScript patterns including type inference, Zod validation at boundaries, and compile-time safety. This is reference material — consult during implementation for TypeScript conventions.
---

# Writing TypeScript

Language-level TypeScript patterns for React + Next.js projects.

## Prefer Type Inference Over Explicit Types

Let TypeScript infer return types. Explicit types add noise.

```typescript
// Good: Inferred
const getCountryName = (code: string) => {
    return COUNTRY_MAP[code] ?? "Unknown";
};

// Bad: Redundant annotation
const getCountryName = (code: string): string => {
    return COUNTRY_MAP[code] ?? "Unknown";
};
```

Why: Less code to maintain. TypeScript catches mismatches automatically.

## No Type Assertions or `any`

```typescript
// Bad
const config = response.data as CountryConfig;
const value: any = getUnknownThing();

// Good: Validate at boundary
const config = CountryConfigSchema.parse(response.data);

// Good: Use unknown + type guard
const value: unknown = getUnknownThing();
if (isCountryConfig(value)) {
    // TypeScript narrows the type
}
```

Why: Assertions bypass the type system. `any` disables it entirely.

## Validate External Data with Zod at Boundaries

Validate once at entry point, then trust types internally.

```typescript
import { z } from "zod";

// Schema for API response
const CatalogResponseSchema = z.object({
    items: z.array(z.object({
        id: z.string(),
        name: z.string(),
        price: z.number().positive(),
    })),
    total: z.number(),
});

type CatalogResponse = z.infer<typeof CatalogResponseSchema>;

// Validate at boundary
export const fetchCatalog = async (country: string): Promise<CatalogResponse> => {
    const response = await hydraClient.get(`/catalog/${country}`);
    return CatalogResponseSchema.parse(response.data); // Throws if invalid
};

// Internal function trusts types — no runtime checks needed
const formatCatalogItem = (item: CatalogResponse["items"][number]) => {
    return `${item.name}: $${item.price}`;
};
```

Why: Zod validates once at the boundary. Internal code trusts the type system.

## Use Discriminated Unions for Type Narrowing

```typescript
type EnrollmentResult =
    | { success: true; orderId: string; redirectUrl: string }
    | { success: false; error: string; retryable: boolean };

const handleResult = (result: EnrollmentResult) => {
    if (result.success) {
        // TypeScript knows orderId exists
        redirect(result.redirectUrl);
    } else {
        // TypeScript knows error exists
        if (result.retryable) showRetryPrompt(result.error);
    }
};
```

Why: Discriminated unions make impossible states unrepresentable.

## Use Named Object Parameters

```typescript
// Bad: Positional args — what do these mean?
createEnrollment("US", "en", true, false, 3);

// Good: Named params — self-documenting
createEnrollment({
    country: "US",
    language: "en",
    isReferral: true,
    skipVerification: false,
    maxRetries: 3,
});
```

Why: Named params are self-documenting and order-independent. Use for 3+ params.

## No Magic Strings or Numbers

```typescript
// Bad
if (country === "US") { ... }
if (step > 4) { ... }

// Good
const MARKETS = { US: "US", JP: "JP", MX: "MX" } as const;
if (country === MARKETS.US) { ... }

const MAX_ENROLLMENT_STEPS = 5;
if (step >= MAX_ENROLLMENT_STEPS) { ... }
```

Why: Constants are searchable, refactorable, and self-documenting.

## Simplify Complex Conditionals

```typescript
// Bad: Nested ternaries
const label = isUS ? "State" : isCA ? "Province" : isMX ? "Estado" : "Region";

// Good: Object lookup
const ADDRESS_LABELS: Record<string, string> = {
    US: "State",
    CA: "Province",
    MX: "Estado",
};
const label = ADDRESS_LABELS[country] ?? "Region";
```

Why: Object lookups are easier to read and extend.

## Derive Types from Const Arrays

```typescript
// Types + runtime values from one source
const ENROLLMENT_STEPS = ["account", "products", "shipping", "payment", "agreements"] as const;
type EnrollmentStep = (typeof ENROLLMENT_STEPS)[number];

// Can iterate AND type-check
ENROLLMENT_STEPS.forEach((step) => validateStep(step));

// Record ensures exhaustive mapping
const STEP_LABELS: Record<EnrollmentStep, string> = {
    account: "Create Account",
    products: "Select Products",
    shipping: "Shipping",
    payment: "Payment",
    agreements: "Agreements",
};
```

Why: Const arrays are both types and runtime values. Record catches missing cases at compile time.

## Environment Variables

```typescript
// Always access through a validated config, never raw process.env
// See env.example for available variables

// Bad
const url = process.env.NEXT_PUBLIC_DEPLOY_URL;

// Good: Centralized, validated
import { appConfig } from "utils/config";
const url = appConfig.deployUrl;
```

Why: Centralized config catches missing env vars at startup, not at runtime.

## Checklist

- [ ] Return types inferred (not annotated)
- [ ] No `any` — use `unknown` + type guards
- [ ] No type assertions (`as X`) — validate with Zod at boundaries
- [ ] External data validated with Zod schemas
- [ ] Types inferred from schemas (`z.infer<typeof schema>`)
- [ ] Discriminated unions for variant types
- [ ] Named object params for functions with 3+ args
- [ ] No magic strings/numbers — use constants
- [ ] Complex conditionals use object lookups, not nested ternaries
- [ ] Const arrays with derived types for exhaustive mappings
