---
name: react-best-practices
description: Comprehensive React and Next.js performance optimization guide with 40+ rules for eliminating waterfalls, optimizing bundles, and improving rendering. Use when optimizing React apps, reviewing performance, or refactoring components.
version: 1.0.0
author: Vercel Engineering
license: MIT
---

# React Best Practices - Performance Optimization

Comprehensive performance optimization guide for React and Next.js applications with 40+ rules organized by impact level.

## When to use this skill

**Use React Best Practices when:**
- Optimizing React or Next.js application performance
- Reviewing code for performance improvements
- Refactoring existing components for better performance
- Implementing new features with performance in mind
- Debugging slow rendering or loading issues
- Reducing bundle size
- Eliminating request waterfalls

## Quick reference

### Critical priorities

1. **Defer await until needed** - Move awaits into branches where they're used
2. **Use Promise.all()** - Parallelize independent async operations
3. **Avoid barrel imports** - Import directly from source files
4. **Dynamic imports** - Lazy-load heavy components
5. **Strategic Suspense** - Stream content while showing layout

### Common patterns

**Parallel data fetching:**
```typescript
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
])
```

**Direct imports:**
```tsx
// BAD - Loads entire library
import { Check } from 'lucide-react'

// GOOD - Loads only what you need
import Check from 'lucide-react/dist/esm/icons/check'
```

**Dynamic components:**
```tsx
import dynamic from 'next/dynamic'

const MonacoEditor = dynamic(
  () => import('./monaco-editor'),
  { ssr: false }
)
```

---

## 1. Eliminating Waterfalls (CRITICAL)

Waterfalls are the #1 performance killer. Each sequential await adds full network latency.

### 1.1 Defer await until needed

```typescript
// BAD - Blocks even if not needed
async function getPage(slug: string) {
  const user = await getUser()
  const page = await getPageBySlug(slug)
  
  if (page.isPublic) {
    return page  // user was fetched but never used
  }
  return page.userId === user.id ? page : null
}

// GOOD - Only await when needed
async function getPage(slug: string) {
  const userPromise = getUser()  // Start fetching
  const page = await getPageBySlug(slug)
  
  if (page.isPublic) {
    return page
  }
  const user = await userPromise  // Only await if needed
  return page.userId === user.id ? page : null
}
```

### 1.2 Use Promise.all for independent operations

```typescript
// BAD - Sequential (3x latency)
const user = await fetchUser()
const posts = await fetchPosts()
const comments = await fetchComments()

// GOOD - Parallel (1x latency)
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
])
```

### 1.3 Strategic Suspense boundaries

```tsx
// GOOD - Stream content while showing layout
export default function Page() {
  return (
    <Layout>
      <Suspense fallback={<HeaderSkeleton />}>
        <Header />
      </Suspense>
      <Suspense fallback={<ContentSkeleton />}>
        <Content />
      </Suspense>
    </Layout>
  )
}
```

---

## 2. Bundle Size Optimization (CRITICAL)

Reducing initial bundle size improves Time to Interactive and LCP.

### 2.1 Avoid barrel file imports

```tsx
// BAD - Loads entire icon library (thousands of icons)
import { Check } from 'lucide-react'

// GOOD - Loads only the icon you need
import Check from 'lucide-react/dist/esm/icons/check'

// BAD - Loads all utilities
import { formatDate } from '@/utils'

// GOOD - Direct import
import { formatDate } from '@/utils/date'
```

### 2.2 Dynamic imports for heavy components

```tsx
// BAD - Loads Monaco in initial bundle
import { MonacoEditor } from './monaco-editor'

// GOOD - Loads Monaco only when needed
import dynamic from 'next/dynamic'

const MonacoEditor = dynamic(
  () => import('./monaco-editor'),
  { 
    ssr: false,
    loading: () => <EditorSkeleton />
  }
)
```

### 2.3 Conditional module loading

```tsx
// GOOD - Only load analytics in production
if (process.env.NODE_ENV === 'production') {
  import('./analytics').then(({ init }) => init())
}
```

### 2.4 Preload on user intent

```tsx
// GOOD - Preload heavy component on hover
const MonacoEditor = dynamic(() => import('./monaco-editor'))

function EditorButton() {
  const preload = () => {
    import('./monaco-editor')
  }
  
  return (
    <button 
      onMouseEnter={preload}
      onClick={() => setShowEditor(true)}
    >
      Open Editor
    </button>
  )
}
```

---

## 3. Re-render Optimization (MEDIUM)

### 3.1 Defer state reads to usage point

```tsx
// BAD - Parent re-renders on every count change
function Parent() {
  const [count, setCount] = useState(0)
  return (
    <div>
      <ExpensiveComponent />
      <span>{count}</span>
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  )
}

// GOOD - Only Counter re-renders
function Parent() {
  return (
    <div>
      <ExpensiveComponent />
      <Counter />
    </div>
  )
}

function Counter() {
  const [count, setCount] = useState(0)
  return (
    <>
      <span>{count}</span>
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </>
  )
}
```

### 3.2 Use transitions for non-urgent updates

```tsx
import { useTransition } from 'react'

function Search() {
  const [query, setQuery] = useState('')
  const [isPending, startTransition] = useTransition()
  
  const handleChange = (e) => {
    setQuery(e.target.value)  // Urgent: update input
    startTransition(() => {
      // Non-urgent: can be interrupted
      setFilteredResults(filterResults(e.target.value))
    })
  }
  
  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending ? <Spinner /> : <Results />}
    </>
  )
}
```

### 3.3 Lazy state initialization

```tsx
// BAD - computeExpensiveValue runs on every render
const [state, setState] = useState(computeExpensiveValue())

// GOOD - Only runs once
const [state, setState] = useState(() => computeExpensiveValue())
```

---

## 4. Server-Side Performance (HIGH)

### 4.1 Use React.cache for per-request deduplication

```tsx
import { cache } from 'react'

// Deduplicated within single request
const getUser = cache(async (userId: string) => {
  return await db.user.findUnique({ where: { id: userId } })
})

// Called multiple times in different components = 1 DB query
async function Header() {
  const user = await getUser(userId)
  // ...
}

async function Sidebar() {
  const user = await getUser(userId)  // Same request, cached
  // ...
}
```

### 4.2 Minimize serialization at RSC boundaries

```tsx
// BAD - Serializes entire user object
async function Page() {
  const user = await getUser()
  return <ClientComponent user={user} />
}

// GOOD - Only pass what's needed
async function Page() {
  const user = await getUser()
  return <ClientComponent userName={user.name} userAvatar={user.avatar} />
}
```

---

## 5. JavaScript Performance (LOW-MEDIUM)

### 5.1 Use Set/Map for O(1) lookups

```typescript
// BAD - O(n) lookup
const ids = [1, 2, 3, 4, 5]
if (ids.includes(targetId)) { ... }

// GOOD - O(1) lookup
const ids = new Set([1, 2, 3, 4, 5])
if (ids.has(targetId)) { ... }
```

### 5.2 Use toSorted() instead of sort()

```typescript
// BAD - Mutates original array
const sorted = items.sort((a, b) => a.name.localeCompare(b.name))

// GOOD - Returns new array
const sorted = items.toSorted((a, b) => a.name.localeCompare(b.name))
```

### 5.3 Hoist static objects outside components

```tsx
// BAD - New object on every render
function Component() {
  return <div style={{ color: 'red', fontSize: 16 }}>Hello</div>
}

// GOOD - Same reference
const styles = { color: 'red', fontSize: 16 }
function Component() {
  return <div style={styles}>Hello</div>
}
```

### 5.4 Cache repeated function calls

```typescript
// BAD - Calls expensive function multiple times
items.forEach(item => {
  if (item.value > getThreshold()) { ... }
})

// GOOD - Cache the result
const threshold = getThreshold()
items.forEach(item => {
  if (item.value > threshold) { ... }
})
```

---

## Key metrics to track

- **Time to Interactive (TTI)**: When page becomes fully interactive
- **Largest Contentful Paint (LCP)**: When main content is visible
- **First Input Delay (FID)**: Responsiveness to user interactions
- **Cumulative Layout Shift (CLS)**: Visual stability
- **Bundle size**: Initial JavaScript payload

---

## Common pitfalls to avoid

**DON'T:**
- Use barrel imports from large libraries
- Block parallel operations with sequential awaits
- Re-render entire trees when only part needs updating
- Load analytics/tracking in the critical path
- Mutate arrays with `.sort()` instead of `.toSorted()`
- Create RegExp or heavy objects inside render

**DO:**
- Import directly from source files
- Use `Promise.all()` for independent operations
- Memoize expensive components
- Lazy-load non-critical code
- Use immutable array methods
- Hoist static objects outside components

---

## Resources

- [React Documentation](https://react.dev)
- [Next.js Documentation](https://nextjs.org)
- [Vercel Bundle Optimization](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)
