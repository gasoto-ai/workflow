---
name: organizing-files
type: pattern
description: Enforces flat file structure with descriptive naming. This is reference material — consult during implementation for file organization conventions.
---

# File Structure & Organization

Flat over nested. Self-describing names. Consistency over cleverness.

## Naming Conventions

| Type | Pattern | Example |
|---|---|---|
| Components | PascalCase | `ProductSelection.tsx` |
| Hooks | camelCase, `use` prefix | `useFilteredProducts.ts` |
| Query hooks | camelCase, `use` prefix + `Query`/`Mutation` | `useCatalogQuery.ts` |
| Utils/helpers | camelCase | `formatCurrency.ts` |
| Services | camelCase | `getMe.ts`, `getCatalog.ts` |
| Types | PascalCase | `EnrollmentTypes.ts` |
| Constants | camelCase file, UPPER_SNAKE values | `appConfig.ts` -> `SUPPORTED_LOCALES` |
| Tests | Same name + `.test` | `ProductSelection.test.tsx` |
| Schemas (Zod) | camelCase + `Schema` suffix | `createAccountSchema.ts` |

## Folder Rules

- **kebab-case** for new folder names. Legacy folders with different conventions keep their existing names — only rename during a major refactor of that area
- **One component per file** — no barrel exports with multiple components
- **Colocate tests** — `Component.test.tsx` next to `Component.tsx`
- **Max 2 levels deep** — if you need deeper nesting, flatten

## Typical Structure (Next.js project)

```
src/
├── app/                    # Pages and API routes — Next.js conventions
├── components/
│   ├── pages/              # Page-specific components (bound to one route)
│   └── shared/             # Reusable across pages
├── queries/                # React Query hooks (useCatalogQuery.ts, useMeQuery.ts)
├── services/               # API service functions (one function per file)
├── hooks/                  # Custom hooks
├── providers/              # React providers (AppProviders, QueryClient, etc.)
├── utils/                  # General utilities
├── constants/              # App-wide constants
├── types/                  # Shared TypeScript types
└── styles/                 # Styles
```

## Where New Files Go

| File Type | Location |
|---|---|
| New page | `src/app/{route}/page.tsx` |
| Page-specific component | `src/components/pages/{PageName}/` |
| Shared component | `src/components/shared/{ComponentName}/` |
| React Query hook | `src/queries/` (`use{Entity}Query.ts`) |
| Custom hook | `src/hooks/` |
| Service function | `src/services/` (one function per file) |
| Utility function | `src/utils/` |
| Type definitions | `src/types/` |
| Zod schema | Next to the component/form that uses it |
| API route handler | `src/app/api/{endpoint}/route.ts` |
| Constants | `src/constants/` |

## Anti-Patterns

```
# Bad: Deep nesting
src/components/shared/forms/inputs/text/variants/TextInput.tsx

# Good: Flat
src/components/shared/TextInput/TextInput.tsx

# Bad: Index file re-exports everything
src/components/shared/index.ts  // exports 38 components

# Good: Import directly
import { TextInput } from "Components/shared/TextInput/TextInput";

# Bad: Generic names
src/utils/helpers.ts
src/utils/utils.ts
src/components/shared/Component.tsx

# Good: Descriptive names
src/utils/formatCurrency.ts
src/utils/distanceCalculator.ts
src/components/shared/AddressForm/AddressForm.tsx
```

## When Adding New Feature Code

1. Check if a similar pattern already exists in the codebase
2. Place files according to the table above
3. Keep related files close together
4. Don't create new top-level directories without discussing first

## Checklist

- [ ] File name matches its primary export
- [ ] Tests colocated next to source file
- [ ] Max 2 levels of folder nesting
- [ ] No barrel index files with mass re-exports
- [ ] kebab-case for new folders, PascalCase components, camelCase utils
- [ ] New files placed in appropriate existing directory
- [ ] No generic file names (`helpers.ts`, `utils.ts`, `index.ts` with multiple exports)
- [ ] Query hooks in `src/queries/` with `use{Entity}Query.ts` naming
- [ ] Service functions in `src/services/` (one per file)
