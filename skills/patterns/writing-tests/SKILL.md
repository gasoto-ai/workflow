---
name: writing-tests
type: pattern
description: Enforces Jest and Testing Library patterns for unit and integration tests. This is reference material — consult during TDD cycles for test-writing conventions.
---

# Writing Tests

Jest + Testing Library patterns for React projects. This is the **toolbox** — for test methodology (red-green-refactor, vertical slices, what to mock), see the `tdd` skill first.

## Test Structure

```tsx
import { render, screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { ProductSelection } from "./ProductSelection";

describe("ProductSelection", () => {
    const defaultProps = {
        products: mockProducts,
        onSelect: jest.fn(),
    };

    beforeEach(() => {
        jest.clearAllMocks();
    });

    test("renders product list with prices", () => {
        render(<ProductSelection {...defaultProps} />);

        expect(screen.getByTestId("product-list")).toBeInTheDocument();
        expect(screen.getByText("Unimate")).toBeInTheDocument();
    });

    test("calls onSelect when product is clicked", async () => {
        const user = userEvent.setup();
        render(<ProductSelection {...defaultProps} />);

        await user.click(screen.getByTestId("product-select-unimate-button"));

        expect(defaultProps.onSelect).toHaveBeenCalledWith("unimate");
    });
});
```

## Selector Strategy

Priority order for finding elements:

```tsx
// 1. data-testid (preferred for all interactive elements)
screen.getByTestId("login-submit-button");

// 2. Role (for accessible elements)
screen.getByRole("button", { name: "Submit" });

// 3. Label text (for form fields)
screen.getByLabelText("Email");

// 4. Text content (for display text only)
screen.getByText("Welcome back");
```

Never use CSS selectors or class names.

## Testing Async Operations

```tsx
test("shows loading state then results", async () => {
    const user = userEvent.setup();
    render(<SearchForm />);

    await user.type(screen.getByTestId("search-input"), "unimate");
    await user.click(screen.getByTestId("search-submit-button"));

    expect(screen.getByTestId("search-loading")).toBeInTheDocument();

    await waitFor(() => {
        expect(screen.getByTestId("search-results")).toBeInTheDocument();
    });

    expect(screen.queryByTestId("search-loading")).not.toBeInTheDocument();
});
```

## Mocking Patterns

> **Philosophy:** See `tdd` skill for when and why to mock. The rule: mock at **system boundaries** only (external APIs, framework APIs you don't control). Don't mock your own modules or internal collaborators. The patterns below show *how* to mock in Jest when boundary mocking is needed.

### Mock Service Functions (System Boundary — External API)

```tsx
// Good: Mock the service function (boundary between app and external API)
jest.mock("services/api/getProducts", () => ({
    getProducts: jest.fn().mockResolvedValue({ items: mockProducts }),
}));

// Good: Mock apiFetch for broader boundary mocking
jest.mock("services/api/apiFetch", () => ({
    apiFetch: jest.fn().mockResolvedValue(mockData),
}));

// Bad: Mocking useQuery internals (not a boundary — it's our code)
jest.mock("@tanstack/react-query", () => ({
    useQuery: () => ({ data: mockData, isLoading: false }),
}));
```

### Mock Next.js (Framework Boundary)

These are framework APIs we don't control — valid boundary mocks:

```tsx
jest.mock("next/navigation", () => ({
    useRouter: () => ({
        push: jest.fn(),
        back: jest.fn(),
    }),
    useSearchParams: () => new URLSearchParams("lang=en"),
    redirect: jest.fn(),
}));

jest.mock("next/headers", () => ({
    cookies: () => ({
        get: jest.fn((name: string) => {
            const values: Record<string, string> = { country: "US", language: "en" };
            return values[name] ? { value: values[name] } : undefined;
        }),
    }),
}));
```

## Testing React Query

### Test Query Client Setup

```tsx
// src/test-utils/createTestQueryClient.ts
import { QueryClient } from "@tanstack/react-query";

export const createTestQueryClient = () =>
    new QueryClient({
        defaultOptions: {
            queries: {
                retry: false, // Don't retry in tests
                gcTime: Infinity, // Don't garbage collect during test
            },
        },
    });
```

### Render With Query Provider

```tsx
// src/test-utils/renderWithProviders.tsx
import { render } from "@testing-library/react";
import { QueryClientProvider } from "@tanstack/react-query";
import { TranslationContext } from "contexts/TranslationContext";
import { createTestQueryClient } from "./createTestQueryClient";

const renderWithProviders = (
    ui: React.ReactElement,
    { translations = mockTranslations, ...options } = {}
) => {
    const queryClient = createTestQueryClient();

    const Wrapper = ({ children }: { children: React.ReactNode }) => (
        <QueryClientProvider client={queryClient}>
            <TranslationContext.Provider value={{ translations, language: "en" }}>
                {children}
            </TranslationContext.Provider>
        </QueryClientProvider>
    );

    return { ...render(ui, { wrapper: Wrapper, ...options }), queryClient };
};

export { renderWithProviders };
```

### Testing Components That Use Queries

```tsx
import { renderWithProviders } from "test-utils/renderWithProviders";
import { getProducts } from "services/api/getProducts";

jest.mock("services/api/getProducts");

test("displays products from catalog query", async () => {
    (getProducts as jest.Mock).mockResolvedValue({ items: mockProducts });

    renderWithProviders(<ProductSelection country="US" />);

    // Wait for query to resolve
    await waitFor(() => {
        expect(screen.getByTestId("product-list")).toBeInTheDocument();
    });

    expect(screen.getByText("Unimate")).toBeInTheDocument();
});
```

### Testing Mutations

```tsx
import { submitOrder } from "services/api/submitOrder";

jest.mock("services/api/submitOrder");

test("submits enrollment and redirects on success", async () => {
    const mockPush = jest.fn();
    jest.spyOn(require("next/navigation"), "useRouter").mockReturnValue({ push: mockPush });
    (submitOrder as jest.Mock).mockResolvedValue({ orderId: "123" });

    renderWithProviders(<ContactForm />);

    // Fill form and submit...
    await userEvent.click(screen.getByTestId("form-submit-button"));

    await waitFor(() => {
        expect(mockPush).toHaveBeenCalledWith("/thank-you");
    });
});

test("shows error message on enrollment failure", async () => {
    (submitOrder as jest.Mock).mockRejectedValue(
        new ApiError(409, "Email already registered")
    );

    renderWithProviders(<ContactForm />);

    // Fill form and submit...
    await userEvent.click(screen.getByTestId("form-submit-button"));

    await waitFor(() => {
        expect(screen.getByTestId("form-error-message")).toBeInTheDocument();
    });
});
```

## Name Tests by Business Scenario

```tsx
// Good: Business scenario
test("prevents enrollment with empty cart", async () => { ... });
test("applies referral discount when code is valid", async () => { ... });
test("redirects to login when session expires", async () => { ... });

// Bad: Implementation detail
test("throws error when items array length is 0", async () => { ... });
test("calls setDiscount with 10 percent", async () => { ... });
```

## Testing Hooks

```tsx
import { renderHook, act } from "@testing-library/react";
import { useFilteredProducts } from "./useFilteredProducts";

test("filters products by country", () => {
    const { result } = renderHook(() =>
        useFilteredProducts(mockProducts, "JP")
    );

    expect(result.current).toHaveLength(3);
    expect(result.current.every(p => p.availableIn.includes("JP"))).toBe(true);
});
```

## What NOT to Test

- TypeScript type constraints (compiler guarantees)
- Simple prop passing (parent renders child with props)
- Third-party library behavior (MUI, RHF internals, React Query internals)
- CSS styling / layout
- Exact snapshot matches (fragile, low value)

## Inline Test Values

```tsx
// Bad: Extra variable
const expectedCount = 3;
const result = getProductCount(mockCart);
expect(result).toBe(expectedCount);

// Good: Inline
expect(getProductCount(mockCart)).toBe(3);
```

## Checklist

- [ ] Tests colocated with source: `ComponentName.test.tsx`
- [ ] `data-testid` used for interactive element selection
- [ ] `userEvent.setup()` used (not `fireEvent`)
- [ ] Async operations use `waitFor` or `findBy*`
- [ ] Mocks cleared in `beforeEach`
- [ ] Test names describe business scenarios, not implementation
- [ ] No snapshot tests for complex components
- [ ] Coverage meets 80% threshold (branches, functions, lines, statements)
- [ ] Wrapper utility used for components needing providers
- [ ] No testing of TypeScript compiler guarantees
- [ ] React Query components use `renderWithProviders` with QueryClientProvider
- [ ] Only boundary mocks: external API services, Next.js framework, external APIs (see `tdd` skill)
- [ ] No mocking of internal modules, hooks, or collaborators you control
- [ ] Mutation tests verify both success and error paths
