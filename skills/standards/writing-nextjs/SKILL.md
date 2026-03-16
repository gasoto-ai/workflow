---
name: writing-nextjs
type: pattern
description: Enforces Next.js 15 App Router patterns for pages, layouts, data fetching, server actions, and API routes. This is reference material — consult during implementation for Next.js conventions.
---

# Writing Next.js (App Router — Next.js 15)

Patterns for Next.js 15 App Router including page structure, data fetching, server components, and API routes.

## App Directory Structure

```
app/
├── layout.tsx          # Root layout (providers, global styles)
├── page.tsx            # Root page (/ route)
├── not-found.tsx       # Custom 404
├── error.tsx           # Error boundary
├── loading.tsx         # Loading UI
├── middleware.ts       # Edge middleware
├── api/                # Route handlers
│   └── health/route.ts
├── home/page.tsx       # /home
├── login/page.tsx      # /login
├── register/page.tsx   # /register
└── [countryCode]/      # Dynamic segments
    └── page.tsx
```

## Server Component (Default)

```tsx
// app/home/page.tsx — Server component by default
import { cookies } from "next/headers";
import fetchTranslations from "utils/fetchTranslations";

export async function generateMetadata({
    searchParams
}: {
    searchParams: Promise<{ lang?: string }>;
}) {
    const params = await searchParams;
    return { title: "Page Title", description: "..." };
}

export default async function HomePage() {
    const cookieStore = await cookies();
    const country = cookieStore.get("country")?.value || "US";
    const { translations } = await fetchTranslations(country, "en");

    return <main>{/* Render with server-fetched data */}</main>;
}
```

## Client Component

```tsx
"use client";

import { useState } from "react";

const InteractiveForm = ({ initialData }: Props) => {
    const [value, setValue] = useState(initialData);
    return <form>{/* Interactive form */}</form>;
};

export default InteractiveForm;
```

## Page Patterns

### Static Page (No Dynamic Data)

```tsx
export default function AboutPage() {
    return <main>Static content</main>;
}
```

### Dynamic Page (Cookies, Headers, searchParams)

```tsx
// Using searchParams or cookies makes page dynamic automatically
// NO need for `export const dynamic = "force-dynamic"`
export default async function Page({
    searchParams
}: {
    searchParams: Promise<{ lang?: string }>;
}) {
    const params = await searchParams;
    return <main>{/* Content */}</main>;
}
```

### Force Dynamic — When It's Actually Needed

Use `force-dynamic` only when the page does server-side data fetching that must be fresh on every request AND doesn't already use cookies/searchParams (which make it dynamic automatically).

```tsx
// Needed: Page fetches fresh data but has no dynamic API usage
export const dynamic = "force-dynamic";
export const revalidate = 0;

export default async function Page() {
    const data = await fetchFreshData();
    return <main>{/* Content */}</main>;
}

// NOT needed: Page already uses cookies or searchParams
export default async function Page({ searchParams }) {
    const params = await searchParams; // Already dynamic
    const cookieStore = await cookies(); // Already dynamic
    return <main>{/* Content */}</main>;
}
```

## Suspense + React Query Prefetching

For pages that need server-fetched data available instantly on the client:

```tsx
// app/register/page.tsx — Server component prefetches for client
import { dehydrate, HydrationBoundary, QueryClient } from "@tanstack/react-query";
import { getCatalog } from "Services/Hydra/getCatalog";
import { cookies } from "next/headers";

export default async function RegisterPage() {
    const cookieStore = await cookies();
    const country = cookieStore.get("country")?.value ?? "US";

    const queryClient = new QueryClient();
    await queryClient.prefetchQuery({
        queryKey: ["catalog", country],
        queryFn: () => getCatalog(country),
    });

    return (
        <HydrationBoundary state={dehydrate(queryClient)}>
            <RegisterFlow />
        </HydrationBoundary>
    );
}
```

Client component uses the prefetched data with no loading flash:

```tsx
// RegisterFlow.tsx (client)
"use client";

const RegisterFlow = () => {
    const { data: catalog } = useCatalogQuery(country); // Instant from cache
    return <ProductSelection products={catalog.items} />;
};
```

### Streaming with Suspense

```tsx
import { Suspense } from "react";

export default function Page() {
    return (
        <main>
            <h1>Title</h1>
            <Suspense fallback={<Skeleton />}>
                <SlowDataComponent />
            </Suspense>
        </main>
    );
}
```

## API Route Handlers

```tsx
// app/api/endpoint/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
    const searchParams = request.nextUrl.searchParams;
    const id = searchParams.get("id");

    const data = await fetchData(id);
    return NextResponse.json(data);
}

export async function POST(request: NextRequest) {
    const body = await request.json();

    if (!body.email) {
        return NextResponse.json(
            { error: "Missing email" },
            { status: 400 }
        );
    }

    const result = await processData(body);
    return NextResponse.json(result, { status: 201 });
}
```

## Metadata

### Static Metadata

```tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
    title: "Page Title",
    description: "Page description",
    openGraph: {
        title: "OG Title",
        description: "OG Description",
        images: ["/og-image.png"]
    }
};
```

### Dynamic Metadata

```tsx
export async function generateMetadata({
    searchParams
}: {
    searchParams: Promise<{ lang?: string }>;
}): Promise<Metadata> {
    const params = await searchParams;
    return getLocalizedMetadata({
        url: "/page-url",
        searchParams: params
    });
}
```

## Layouts

```tsx
export default function RootLayout({
    children
}: {
    children: React.ReactNode;
}) {
    return (
        <html lang="en">
            <body>
                <AppProviders>{children}</AppProviders>
            </body>
        </html>
    );
}
```

## Loading & Error States

```tsx
// app/loading.tsx
export default function Loading() {
    return <div>Loading...</div>;
}

// app/error.tsx (must be client component)
"use client";

export default function Error({
    error,
    reset
}: {
    error: Error;
    reset: () => void;
}) {
    return (
        <div>
            <h2>Something went wrong</h2>
            <button onClick={reset}>Try again</button>
        </div>
    );
}
```

## Middleware

```tsx
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
    const { pathname } = request.nextUrl;
    const token = request.cookies.get("_unicityToken_v5_enroll");

    if (pathname.startsWith("/register") && !token) {
        return NextResponse.redirect(new URL("/login", request.url));
    }

    const response = NextResponse.next();
    response.headers.set("X-Frame-Options", "DENY");
    return response;
}

export const config = {
    matcher: ["/((?!api|_next/static|_next/image|favicon.ico).*)"]
};
```

## Server Actions

```tsx
"use server";

export async function submitForm(formData: FormData) {
    const email = formData.get("email") as string;
    if (!email) throw new Error("Email required");

    const result = await saveToDatabase({ email });
    return result;
}
```

## Dynamic Imports (Client Components)

```tsx
import dynamic from "next/dynamic";

const HeavyChart = dynamic(() => import("@/components/HeavyChart"), {
    loading: () => <Skeleton className="h-64" />,
    ssr: false
});
```

## Checklist

- [ ] Server components by default — only add "use client" when needed
- [ ] Metadata uses `generateMetadata` or static `metadata` export
- [ ] `force-dynamic` only when truly needed (not when cookies/searchParams already make it dynamic)
- [ ] API routes use Route Handlers (`route.ts`), not Pages API routes
- [ ] `searchParams` and `params` are awaited (Next.js 15 async APIs)
- [ ] Loading/error states handled with `loading.tsx` / `error.tsx`
- [ ] Heavy client components lazy loaded with `dynamic()`
- [ ] Images use `next/image` with proper dimensions
- [ ] Security headers set in middleware
- [ ] React Query prefetch + HydrationBoundary for server-to-client data handoff
- [ ] Suspense boundaries wrap slow async components
