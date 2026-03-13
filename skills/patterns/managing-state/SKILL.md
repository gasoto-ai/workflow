---
name: managing-state
type: pattern
description: Enforces state management patterns using React Query for server state, RHF for forms, and slim context for auth. This is reference material — consult during implementation for state management conventions.
---

# Managing State

State belongs in exactly one place. Pick the right tool for each type.

## The Golden Rule — Where State Lives

| State Type | Tool | Example |
|---|---|---|
| Async server data (client) | React Query | Product catalog, user profile, order status |
| Server data (server) | Server component fetch | Translations, OG metadata, initial page data |
| Form state | React Hook Form | Account fields, shipping address, payment details |
| Auth state | Slim UserContext | Auth token, logged-in user info |
| UI state | Local `useState` | Modals open/closed, accordion expanded, loading |
| URL state | `searchParams` / cookies | Language, country, referral code |

**Never mix these.** Form data doesn't go in context. Server data doesn't go in useState. No Zustand, Redux, or Jotai.

## React Query for Client-Side Server State

When a client component needs async server data, use React Query — not `useEffect` + `useState`, not context.

```tsx
// GOOD: React Query manages loading, caching, refetching
const { data: catalog, isLoading, error } = useCatalogQuery(country);

if (isLoading) return <Skeleton />;
if (error) return <ErrorMessage error={error} />;
return <ProductList products={catalog.items} />;
```

```tsx
// BAD: Manual fetch in useEffect
const [products, setProducts] = useState([]);
const [isLoading, setIsLoading] = useState(false);
useEffect(() => {
    setIsLoading(true);
    fetchProducts(country).then(setProducts).finally(() => setIsLoading(false));
}, [country]);
```

See `data-fetching` skill for full React Query patterns.

## Current State (Legacy) -> Target State

### OrderContext (Remove)

```tsx
// BAD (current): OrderContext holds everything
const { orderData, setOrderData } = useOrderContext();
setOrderData({ ...orderData, firstName: "John" });

// GOOD (target): RHF holds form state
const { register } = useFormContext();
<input {...register("firstName")} />;
```

### ProductContext (Replace with React Query)

```tsx
// BAD: Storing fetched products in context
const { products, setProducts } = useProductContext();
useEffect(() => {
    fetchProducts(country).then(setProducts);
}, [country]);

// GOOD (server component): Fetch and pass as props
// app/home/page.tsx
const products = await getCatalog(country);
return <ProductSelection products={products} />;

// GOOD (client component): React Query
const { data: products } = useCatalogQuery(country);
```

### UserContext (Slim Down)

```tsx
// BAD: UserContext holds cart, preferences, order history
const { user, cart, preferences, orderHistory } = useUserContext();

// GOOD: UserContext is auth-only
const { user, isAuthenticated, token } = useUserContext();
// Cart -> RHF, preferences -> cookies, history -> React Query
```

## Local State — Keep It Local

```tsx
// Good: UI state stays in the component
const [isModalOpen, setIsModalOpen] = useState(false);
const [activeTab, setActiveTab] = useState("products");

// Bad: UI state in context
const { isModalOpen, setIsModalOpen } = useUIContext(); // Over-engineered
```

Why: If only one component uses the state, it belongs in that component.

## Frequently Changing State — Still Local

```tsx
// GOOD: Keep frequently changing state local, compose with props
const NavigationMenu = () => {
    const [isOpen, setIsOpen] = useState(false);
    const [activeValue, setActiveValue] = useState("");
    return (
        <nav>
            <MenuButton isOpen={isOpen} onToggle={() => setIsOpen(!isOpen)} />
            <MenuContent value={activeValue} onChange={setActiveValue} />
        </nav>
    );
};

// BAD: Context for frequently changing values (re-renders everything)
type NavigationMenuContextType = {
    isNavigationMenuOpen: boolean;
    navigationMenuValue: string;
};
```

Why: Local state + composition avoids unnecessary re-renders without adding a library.

## URL State for Shareable Values

```tsx
// Good: Language and country in URL/cookies — shareable, bookmarkable
// /home?lang=es -> Spanish version
// Cookie: country=MX -> Mexico market

// Bad: Language in React state
const [language, setLanguage] = useState("en"); // Lost on refresh
```

## Derive, Don't Store

```tsx
// Bad: Storing computed values
const [totalPrice, setTotalPrice] = useState(0);
useEffect(() => {
    setTotalPrice(items.reduce((sum, item) => sum + item.price * item.qty, 0));
}, [items]);

// Good: Compute during render
const totalPrice = items.reduce((sum, item) => sum + item.price * item.qty, 0);

// Good: Memoize if expensive
const totalPrice = useMemo(
    () => items.reduce((sum, item) => sum + item.price * item.qty, 0),
    [items]
);
```

## Multi-Step Form State (Migration Target)

```tsx
// The enrollment flow: Account -> Products -> Shipping -> Payment -> Agreements
// All steps share one RHF instance via FormProvider

const EnrollmentForm = () => {
    const methods = useForm({
        defaultValues: {
            firstName: "",
            lastName: "",
            email: "",
            selectedProducts: [],
            address: { line1: "", city: "", state: "", zip: "", country: "" },
        },
    });

    return (
        <FormProvider {...methods}>
            <StepRouter />
        </FormProvider>
    );
};

// Each step accesses shared form state
const AccountStep = () => {
    const { register, formState: { errors } } = useFormContext();
    return (
        <div>
            <input {...register("firstName")} />
            <input {...register("email")} />
        </div>
    );
};
```

## Anti-Patterns

```tsx
// BAD: Syncing context to local state
const { orderData } = useOrderContext();
const [localData, setLocalData] = useState(orderData);
useEffect(() => setLocalData(orderData), [orderData]); // Why??

// BAD: Using context as a form store
const { setOrderData } = useOrderContext();
const handleChange = (e) => {
    setOrderData(prev => ({ ...prev, [e.target.name]: e.target.value }));
};
// This re-renders EVERYTHING that uses OrderContext on every keystroke

// BAD: Duplicating React Query data to context
const { data } = useCatalogQuery(country);
useEffect(() => {
    setProductContext(data); // Don't! Use query data directly
}, [data]);

// BAD: Fetching in useEffect + storing in state
const [products, setProducts] = useState([]);
useEffect(() => {
    fetchProducts().then(setProducts);
}, []);
// Use server components, React Query, or pass as props instead
```

## Checklist

- [ ] Form state managed by RHF — not in contexts
- [ ] Async server data on client uses React Query — not useEffect + useState
- [ ] Server data fetched in server components or passed as props
- [ ] React Query data used directly — not duplicated to context or state
- [ ] UserContext is auth-only (token, user info, isAuthenticated)
- [ ] UI state uses local `useState` — not context
- [ ] URL/cookie state for language, country, referral
- [ ] Derived values computed during render — not stored in state
- [ ] No `useEffect` to sync state between sources
- [ ] Multi-step forms use `FormProvider` with shared `useForm` instance
- [ ] No context re-renders on every keystroke (form data out of context)
- [ ] No Zustand, Redux, or Jotai
