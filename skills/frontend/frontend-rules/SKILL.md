---
name: frontend-rules
description: Frontend development rules for HowTheF* Next.js 15 + TypeScript app. Use when working in howthef-frontend/ or discussing React, state management, TanStack Query, or UI components.
---

# Frontend Rules

## Running
- `npm run dev` - Dev server
- `npm run supabase:gen-types` - Regenerate types

## State Management (Choose in Order)
1. **Server State** → TanStack Query
2. **URL State** → nuqs/URLSearchParams
3. **Form State** → React Hook Form
4. **Global UI State** → Zustand
5. **Complex State** → useReducer
6. **Simple Local** → useState (last resort)

**Anti-patterns:** Never `useState` + `useEffect` for sync. Never store derived state. Never query Supabase directly from client.

## Naming
- Components: `PascalCaseComponent.tsx`
- Services: `camelCaseService.ts`
- Hooks: `useSomethingHook.ts`
- Types: `something.interface.ts`

## File Path Comments
EVERY file starts with: `// app/features/organizations/organization.interface.ts`

## Architecture
```
app/
├── (routes)/              # Next.js pages
├── features/{feature}/    # Feature modules
│   ├── components/
│   ├── services_and_hooks/
│   └── {feature}.interface.ts
├── shared/                # Providers, hooks, stores
└── components/            # shadcn/ui components
```

## API Pattern
1. **API Route** - Server-side Supabase query
2. **TanStack Query Hook** - Fetch from API route
3. **Component** - Use hook, render data

## Critical Rules
- NEVER direct Supabase from client - use API routes
- ALWAYS use file path comments
- ALWAYS follow naming conventions
