---
trigger: always_on
---

# Next.js + Zustand + TanStack Query Coding Rules

> These rules are **mandatory**.
> Do not infer intent. Follow exactly.

---

## 1. Responsibility Separation

- **TanStack Query** MUST handle:
  - Server state (API data)
  - Data fetching, caching, synchronization
  - Background refetching
  - Optimistic updates
  - Pagination & infinite scroll

- **Zustand Stores** MUST handle:
  - Client-side global state (cart, UI preferences)
  - Business logic on client state
  - Data transformation on client state
  - State that needs persistence (localStorage)

- **Zustand Stores** MUST NOT:
  - Fetch server data (use TanStack Query)
  - Contain React components
  - Import from `components/`
  - Handle routing logic directly
  - Manage UI-specific state (use local state)

- **Components** MUST handle:
  - UI rendering
  - User interactions
  - Local UI state (forms, modals, toggles)

- **Server Components** MUST handle:
  - Initial data fetching (SSR)
  - SEO metadata
  - Server-side rendering

### State Type Decision Matrix

| State Type | Use |
|------------|-----|
| Server data (API) | TanStack Query |
| Client global state (cart, user prefs) | Zustand |
| Temporary UI state (modals, forms) | Local State |
| URL state (filters, pagination) | URL params |

---

## 2. Directory Structure

```txt
app/
  [feature]/
    page.tsx              # Server Component (default)
    layout.tsx            # Feature layout (optional)
components/
  layout/                 # Global layout components
  features/               # Feature-specific components
    [feature]/
      ComponentName.tsx   # Client Component
  ui/                     # shadcn/ui components
lib/
  store.ts                # Zustand stores (client state)
  queries.ts              # TanStack Query hooks (server state)
  api.ts                  # API client functions
  data.ts                 # Type definitions & mock data
  utils.ts                # Utility functions
```

Rules:

- All Zustand stores **MUST** be in `lib/store.ts` or `lib/stores/`
- All TanStack Query hooks **MUST** be in `lib/queries.ts` or `lib/queries/`
- All API functions **MUST** be in `lib/api.ts` or `lib/api/`
- Feature components **MUST** be in `components/features/[feature]/`
- Server Components **MUST** be in `app/` directory
- Client Components **MUST** have `"use client"` directive

---

## 3. Component Naming

- Component files **MUST** use PascalCase: `MenuCard.tsx`
- Page files **MUST** use lowercase: `page.tsx`, `layout.tsx`, `loading.tsx`
- Store hooks **MUST** start with `use`: `useCartStore`, `useCafeStore`
- Folders **MUST** use kebab-case: `order-history/`, `cafe-list/`

Examples:

- ✅ `components/features/menu/MenuCard.tsx`
- ✅ `app/cafes/[id]/page.tsx`
- ✅ `useCartStore`
- ❌ `components/features/menu/menuCard.tsx`
- ❌ `components/features/Menu/menu-card.tsx`
- ❌ `cartStore` (missing 'use' prefix)

---

## 4. Server vs Client Components

### Server Components (Default)

- **MUST** be used by default in `app/` directory
- **MUST** handle data fetching
- **MUST NOT** use hooks (`useState`, `useEffect`, etc.)
- **MUST NOT** use browser APIs
- **MUST NOT** use event handlers

```tsx
// app/cafes/page.tsx
export default async function CafesPage() {
  // Data fetching on server
  const cafes = await fetchCafes();
  return <CafeList cafes={cafes} />;
}
```

### Client Components

- **MUST** have `"use client"` directive at the top
- **MUST** be used when:
  - Using React hooks
  - Using browser APIs
  - Handling user interactions
  - Using Zustand stores

```tsx
"use client";

import { useCartStore } from "@/lib/store";

export function CartButton() {
  const totalItems = useCartStore((state) => state.totalItems());
  return <button>Cart ({totalItems})</button>;
}
```

---

## 5. Zustand Store Pattern

### Store Structure

- All state **MUST** be private (managed internally)
- Public access **MUST** be through actions or selectors
- Each store **MUST** have a clear, single responsibility

```tsx
interface CartState {
  // State (internal)
  items: CartItem[];

  // Actions (public API)
  addToCart: (item: CartItem) => void;
  removeFromCart: (id: string) => void;

  // Selectors (computed values)
  totalItems: () => number;
  totalPrice: () => number;
}

export const useCartStore = create<CartState>()((set, get) => ({
  items: [],
  addToCart: (item) => set((state) => ({
    items: [...state.items, item]
  })),
  removeFromCart: (id) => set((state) => ({
    items: state.items.filter((i) => i.id !== id)
  })),
  totalItems: () => get().items.reduce((acc, item) => acc + item.quantity, 0),
  totalPrice: () => get().items.reduce((acc, item) => acc + item.price, 0),
}));
```

### Persistence

- Use `persist` middleware for data that should survive page refresh
- **MUST** specify a unique `name` for storage key

```tsx
export const useCartStore = create<CartState>()(
  persist(
    (set, get) => ({
      // ... store implementation
    }),
    {
      name: "cart-storage",
    }
  )
);
```

---

## 6. Store Access Pattern

### Selective Subscription (Preferred)

- **MUST** use selector functions to prevent unnecessary re-renders
- **MUST** subscribe only to needed state slices

```tsx
// ✅ GOOD - Only re-renders when totalItems changes
function CartBadge() {
  const totalItems = useCartStore((state) => state.totalItems());
  return <Badge>{totalItems}</Badge>;
}

// ❌ BAD - Re-renders on any store change
function CartBadge() {
  const store = useCartStore();
  return <Badge>{store.totalItems()}</Badge>;
}
```

### Actions Only

- Use `useShallow` for multiple selections
- Actions don't cause re-renders

```tsx
function ProductCard({ product }: Props) {
  const addToCart = useCartStore((state) => state.addToCart);

  return (
    <button onClick={() => addToCart(product)}>
      Add to Cart
    </button>
  );
}
```

---

## 7. State Management Guidelines

### When to use TanStack Query

- ✅ Fetching data from APIs
- ✅ Caching server responses
- ✅ Background data synchronization
- ✅ Pagination & infinite loading
- ✅ Optimistic UI updates
- ✅ Data invalidation after mutations

### When to use Zustand

- ✅ Client-side global state (cart, user preferences)
- ✅ Shared state across multiple pages (not from server)
- ✅ Data that needs localStorage persistence
- ✅ Complex client-side business logic

### When to use Local State

- ✅ Form inputs
- ✅ Modal open/close
- ✅ Temporary UI state
- ✅ Component-specific toggles

```tsx
"use client";

// ✅ GOOD - Server state with TanStack Query
function CafeList() {
  const { data: cafes, isLoading } = useQuery({
    queryKey: ['cafes'],
    queryFn: fetchCafes,
  });

  if (isLoading) return <Spinner />;
  return <List cafes={cafes} />;
}

// ✅ GOOD - Client global state with Zustand
function CartButton() {
  const totalItems = useCartStore((state) => state.totalItems());
  return <Button>Cart ({totalItems})</Button>;
}

// ✅ GOOD - Local UI state
function SearchBar() {
  const [query, setQuery] = useState("");
  const [isOpen, setIsOpen] = useState(false);

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
    />
  );
}

// ❌ BAD - Don't fetch server data with Zustand
const useCafeStore = create((set) => ({
  cafes: [],
  fetchCafes: async () => {
    const data = await fetch('/api/cafes');
    set({ cafes: data });
  }
}));

// ❌ BAD - Don't put temporary UI state in Zustand
const useUIStore = create((set) => ({
  searchQuery: "",
  isModalOpen: false,
}));
```

---

## 8. TypeScript Rules

### Type Definitions

- All interfaces **MUST** be exported from `lib/data.ts` or co-located
- Props **MUST** have explicit type definitions
- Store state **MUST** have interface definition

```tsx
// lib/data.ts
export interface Product {
  id: string;
  name: string;
  price: number;
}

// Component
interface ProductCardProps {
  product: Product;
  onAddToCart: (product: Product) => void;
}

export function ProductCard({ product, onAddToCart }: ProductCardProps) {
  return <div>{product.name}</div>;
}
```

### Avoid `any`

- Using `any` type is **FORBIDDEN** unless absolutely necessary
- Use `unknown` when type is truly unknown
- Use generics for reusable components

---

## 9. File Organization

### Imports Order

1. React / Next.js imports
2. Third-party libraries
3. Local components
4. Local utilities / stores
5. Type imports
6. Styles

```tsx
"use client";

import { useState } from "react";
import { useRouter } from "next/navigation";
import { motion } from "framer-motion";

import { Button } from "@/components/ui/button";
import { MenuCard } from "@/components/features/menu/MenuCard";

import { useCartStore } from "@/lib/store";
import { cn } from "@/lib/utils";

import type { Product } from "@/lib/data";
```

### Path Aliases

- **MUST** use `@/` alias for absolute imports
- **MUST NOT** use relative imports beyond parent directory

```tsx
// ✅ GOOD
import { Button } from "@/components/ui/button";
import { useCartStore } from "@/lib/store";

// ❌ BAD
import { Button } from "../../../components/ui/button";
import { useCartStore } from "../../lib/store";
```

---

## 10. Component Pattern

### Composition over Props Drilling

- Use composition to avoid deep prop drilling
- Use Context for deeply nested shared state (theme, i18n)
- Use Zustand for global state

```tsx
// ✅ GOOD - Composition
function ProductPage() {
  return (
    <ProductLayout>
      <ProductHeader />
      <ProductDetails />
      <AddToCartButton />
    </ProductLayout>
  );
}

// ❌ BAD - Props drilling
function ProductPage() {
  const product = useProduct();
  return (
    <ProductLayout product={product}>
      <ProductHeader product={product} />
      <ProductDetails product={product} />
      <AddToCartButton product={product} />
    </ProductLayout>
  );
}
```

---

## 11. Performance Rules

### Code Splitting

- Large components **MUST** use dynamic imports
- Route-level code splitting is automatic with App Router

```tsx
import dynamic from "next/dynamic";

const HeavyComponent = dynamic(
  () => import("@/components/features/HeavyComponent"),
  { loading: () => <Spinner /> }
);
```

### Image Optimization

- **MUST** use `next/image` for all images
- **MUST** specify width and height (or fill)

```tsx
import Image from "next/image";

<Image
  src="/cafe.jpg"
  alt="Cafe"
  width={400}
  height={300}
  priority // for above-the-fold images
/>
```

### Memoization

- Use `useMemo` for expensive calculations only
- Use `useCallback` when passing callbacks to memoized children
- **DO NOT** over-optimize prematurely

---

## 12. Styling Rules

### Tailwind CSS

- **MUST** use Tailwind utility classes
- **MUST** use `cn()` utility for conditional classes
- **MUST NOT** use inline styles unless absolutely necessary

```tsx
import { cn } from "@/lib/utils";

function Button({ variant, className, ...props }: ButtonProps) {
  return (
    <button
      className={cn(
        "px-4 py-2 rounded-lg",
        variant === "primary" && "bg-blue-500 text-white",
        variant === "secondary" && "bg-gray-200 text-gray-900",
        className
      )}
      {...props}
    />
  );
}
```

### Component Variants

- Use `class-variance-authority` for complex variant patterns
- Keep variant logic co-located with component

---

## 13. TanStack Query Pattern

### Setup

- Create `QueryProvider` wrapper in `components/providers/`
- Wrap app with `QueryClientProvider` in root layout

```tsx
// components/providers/query-provider.tsx
"use client";

import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import { useState } from "react";

export function QueryProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000, // 1 minute
            gcTime: 5 * 60 * 1000, // 5 minutes
            retry: 1,
            refetchOnWindowFocus: false,
          },
        },
      })
  );

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

```tsx
// app/layout.tsx
import { QueryProvider } from "@/components/providers/query-provider";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <QueryProvider>
          {children}
        </QueryProvider>
      </body>
    </html>
  );
}
```

### Query Hooks Pattern

- **MUST** define query hooks in `lib/queries.ts`
- **MUST** separate API functions from query hooks
- **MUST** use type-safe query keys

```tsx
// lib/api.ts
export async function fetchCafes(): Promise<Cafe[]> {
  const res = await fetch('/api/cafes');
  if (!res.ok) throw new Error('Failed to fetch cafes');
  return res.json();
}

export async function fetchCafe(id: string): Promise<Cafe> {
  const res = await fetch(`/api/cafes/${id}`);
  if (!res.ok) throw new Error('Failed to fetch cafe');
  return res.json();
}

export async function createCafe(data: CreateCafeInput): Promise<Cafe> {
  const res = await fetch('/api/cafes', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  });
  if (!res.ok) throw new Error('Failed to create cafe');
  return res.json();
}
```

```tsx
// lib/queries.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { fetchCafes, fetchCafe, createCafe } from "./api";
import type { Cafe, CreateCafeInput } from "./data";

// Query Keys (type-safe)
export const cafeKeys = {
  all: ['cafes'] as const,
  lists: () => [...cafeKeys.all, 'list'] as const,
  list: (filters: string) => [...cafeKeys.lists(), { filters }] as const,
  details: () => [...cafeKeys.all, 'detail'] as const,
  detail: (id: string) => [...cafeKeys.details(), id] as const,
};

// Queries
export function useCafes() {
  return useQuery({
    queryKey: cafeKeys.lists(),
    queryFn: fetchCafes,
  });
}

export function useCafe(id: string) {
  return useQuery({
    queryKey: cafeKeys.detail(id),
    queryFn: () => fetchCafe(id),
    enabled: !!id, // Only run if id exists
  });
}

// Mutations
export function useCreateCafe() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createCafe,
    onSuccess: (newCafe) => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: cafeKeys.lists() });

      // Or optimistically update
      queryClient.setQueryData<Cafe[]>(cafeKeys.lists(), (old) =>
        old ? [...old, newCafe] : [newCafe]
      );
    },
  });
}
```

### Usage in Components

```tsx
"use client";

import { useCafes, useCreateCafe } from "@/lib/queries";

function CafeList() {
  const { data: cafes, isLoading, error } = useCafes();
  const createCafe = useCreateCafe();

  if (isLoading) return <Spinner />;
  if (error) return <Error message={error.message} />;

  return (
    <div>
      {cafes?.map((cafe) => (
        <CafeCard key={cafe.id} cafe={cafe} />
      ))}
      <button
        onClick={() => createCafe.mutate({ name: "New Cafe" })}
        disabled={createCafe.isPending}
      >
        Add Cafe
      </button>
    </div>
  );
}
```

### Optimistic Updates

```tsx
export function useUpdateCafe() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updateCafe,
    onMutate: async (updatedCafe) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: cafeKeys.detail(updatedCafe.id) });

      // Snapshot previous value
      const previousCafe = queryClient.getQueryData(cafeKeys.detail(updatedCafe.id));

      // Optimistically update
      queryClient.setQueryData(cafeKeys.detail(updatedCafe.id), updatedCafe);

      // Return context with snapshot
      return { previousCafe };
    },
    onError: (err, updatedCafe, context) => {
      // Rollback on error
      if (context?.previousCafe) {
        queryClient.setQueryData(
          cafeKeys.detail(updatedCafe.id),
          context.previousCafe
        );
      }
    },
    onSettled: (updatedCafe) => {
      // Refetch after mutation
      queryClient.invalidateQueries({ queryKey: cafeKeys.detail(updatedCafe.id) });
    },
  });
}
```

### Pagination

```tsx
export function useCafesPaginated(page: number) {
  return useQuery({
    queryKey: [...cafeKeys.lists(), { page }],
    queryFn: () => fetchCafesPaginated(page),
    placeholderData: keepPreviousData, // Keep old data while fetching
  });
}
```

### Infinite Query

```tsx
export function useCafesInfinite() {
  return useInfiniteQuery({
    queryKey: cafeKeys.lists(),
    queryFn: ({ pageParam = 0 }) => fetchCafesPaginated(pageParam),
    getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
    initialPageParam: 0,
  });
}

// Usage
function CafeInfiniteList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useCafesInfinite();

  return (
    <>
      {data?.pages.map((page) =>
        page.cafes.map((cafe) => <CafeCard key={cafe.id} cafe={cafe} />)
      )}
      {hasNextPage && (
        <button onClick={() => fetchNextPage()} disabled={isFetchingNextPage}>
          Load More
        </button>
      )}
    </>
  );
}
```

### Dependent Queries

```tsx
function CafeProducts({ cafeId }: { cafeId: string }) {
  // First query
  const { data: cafe } = useCafe(cafeId);

  // Dependent query - only runs after cafe is fetched
  const { data: products } = useQuery({
    queryKey: ['products', cafeId],
    queryFn: () => fetchProducts(cafeId),
    enabled: !!cafe, // Only run if cafe exists
  });

  return <ProductList products={products} />;
}
```

### Error Handling

```tsx
function CafeList() {
  const { data, error, isError, refetch } = useCafes();

  if (isError) {
    return (
      <div>
        <p>Error: {error.message}</p>
        <button onClick={() => refetch()}>Retry</button>
      </div>
    );
  }

  return <List cafes={data} />;
}
```

---

## 14. Async Patterns

### Data Fetching in Server Components

```tsx
// app/cafes/[id]/page.tsx
async function CafePage({ params }: { params: { id: string } }) {
  const cafe = await fetchCafe(params.id);

  if (!cafe) {
    notFound();
  }

  return <CafeDetails cafe={cafe} />;
}
```

### Data Fetching in Client Components

- **MUST** use TanStack Query for server state
- **MUST NOT** use `useEffect` for data fetching (use TanStack Query instead)
- Handle loading and error states with query states

```tsx
"use client";

// ✅ GOOD - Use TanStack Query
function CafeList() {
  const { data: cafes, isLoading, error } = useCafes();

  if (isLoading) return <Spinner />;
  if (error) return <Error />;
  return <List cafes={cafes} />;
}

// ❌ BAD - Don't use useEffect for data fetching
function CafeList() {
  const [cafes, setCafes] = useState<Cafe[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchCafes()
      .then(setCafes)
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <Spinner />;
  return <List cafes={cafes} />;
}
```

### Hybrid Pattern (Server + Client)

- Prefetch data in Server Component
- Hydrate in Client Component with TanStack Query

```tsx
// app/cafes/page.tsx (Server Component)
import { HydrationBoundary, QueryClient, dehydrate } from "@tanstack/react-query";
import { CafeListClient } from "./CafeListClient";
import { cafeKeys } from "@/lib/queries";
import { fetchCafes } from "@/lib/api";

export default async function CafesPage() {
  const queryClient = new QueryClient();

  // Prefetch on server
  await queryClient.prefetchQuery({
    queryKey: cafeKeys.lists(),
    queryFn: fetchCafes,
  });

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <CafeListClient />
    </HydrationBoundary>
  );
}
```

```tsx
// app/cafes/CafeListClient.tsx (Client Component)
"use client";

import { useCafes } from "@/lib/queries";

export function CafeListClient() {
  // Data is already prefetched, no loading state
  const { data: cafes } = useCafes();

  return <List cafes={cafes} />;
}
```

---

## 15. Routing Rules

### Navigation

- **MUST** use `next/link` for internal navigation
- **MUST** use `useRouter` hook for programmatic navigation

```tsx
import Link from "next/link";
import { useRouter } from "next/navigation";

function Navigation() {
  const router = useRouter();

  return (
    <>
      <Link href="/cafes">Cafes</Link>
      <button onClick={() => router.push("/cart")}>
        Go to Cart
      </button>
    </>
  );
}
```

### Dynamic Routes

- Use `[param]` for dynamic segments
- Use `[...slug]` for catch-all segments

---

## 16. Error Handling

### Error Boundaries

- Create `error.tsx` for route-level error handling
- Create `global-error.tsx` for app-level errors

```tsx
// app/cafes/error.tsx
"use client";

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

---

## 17. Testing Considerations

- Components **MUST** be testable (pure, no side effects in render)
- Store actions **MUST** be tested independently
- Mock Zustand stores in tests

---

## 18. Rule Priority

1. **Correctness** - Code must work as intended
2. **Type Safety** - Leverage TypeScript fully
3. **Performance** - Optimize when necessary
4. **Readability** - Code should be self-documenting
5. **Convenience** - DX improvements (lowest priority)

---

## 19. Enforcement

- Any code violating these rules **MUST** be rejected
- No exceptions unless explicitly documented
- Rules trump personal preferences

---

## 20. Next.js Specific

### Metadata API

- Use `generateMetadata` for dynamic SEO

```tsx
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const cafe = await fetchCafe(params.id);
  return {
    title: cafe.name,
    description: cafe.description,
  };
}
```

### Route Handlers

- Create `route.ts` for API endpoints
- **MUST** export HTTP method handlers

```tsx
// app/api/cafes/route.ts
export async function GET() {
  const cafes = await fetchCafes();
  return Response.json(cafes);
}
```

---

## 21. Common Pitfalls to Avoid

- ❌ Using Client Components when Server Components suffice
- ❌ Fetching server data with Zustand (use TanStack Query)
- ❌ Using `useEffect` for data fetching (use TanStack Query)
- ❌ Putting entire app state in Zustand (use local state when appropriate)
- ❌ Not invalidating queries after mutations
- ❌ Fetching the same data multiple times (use query cache)
- ❌ Mutating state directly (always use `set()` in Zustand)
- ❌ Subscribing to entire store when only part is needed
- ❌ Creating too many small stores (balance granularity)
- ❌ Ignoring TypeScript errors
- ❌ Over-engineering simple components

---

## Reference

- Next.js: https://nextjs.org/docs
- TanStack Query: https://tanstack.com/query/latest/docs/react/overview
- Zustand: https://docs.pmnd.rs/zustand
- React Server Components: https://react.dev/reference/rsc/server-components
