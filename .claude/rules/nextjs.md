# Règles Next.js 15 App Router

## Server vs Client Components

- **Server Component par défaut** — ne jamais ajouter `"use client"` sans raison explicite
- **`"use client"` SEULEMENT si** : hooks React (`useState`, `useEffect`), event listeners, browser APIs (`window`, `document`, `localStorage`), ou animations
- **Jamais de `useEffect` pour fetcher** — utiliser `async/await` directement dans le Server Component
- **Jamais appeler ses propres Route Handlers depuis un Server Component** — appeler la fonction directement

```tsx
// INTERDIT
useEffect(() => { fetch('/api/tickets').then(setTickets) }, [])

// CORRECT
async function Page() {
  const tickets = await getTickets() // appel direct, pas de fetch
  return <TicketList tickets={tickets} />
}
```

## Next.js 15 — Changements Obligatoires

`params` et `searchParams` sont des **Promises** en Next.js 15 — toujours les `await` :

```tsx
// INTERDIT (pattern Next.js 14)
export default function Page({ params }: { params: { id: string } }) {
  const { id } = params
}

// CORRECT (Next.js 15)
export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
}
```

## Server Actions

- **Validation Zod obligatoire** avec `safeParse()` avant toute opération DB
- **`"use server"`** en tête du fichier ou de la fonction
- **Jamais de Route Handler** si une Server Action suffit

```tsx
// app/actions/tickets.ts
"use server"
import { z } from "zod"
import { prisma } from "@/lib/prisma"
import { auth } from "@/lib/auth"

const CreateTicketSchema = z.object({
  title: z.string().min(1).max(200),
  priority: z.enum(["low", "medium", "high", "critical"]),
  type: z.enum(["bug", "assistance", "feature"]),
  productId: z.string().cuid(),
  clientId: z.string().cuid(),
  description: z.string().min(10),
})

export async function createTicket(formData: FormData) {
  const session = await auth()
  if (!session) return { error: "Unauthorized" }

  const parsed = CreateTicketSchema.safeParse(Object.fromEntries(formData))
  if (!parsed.success) return { error: parsed.error.flatten() }

  return prisma.ticket.create({ data: { ...parsed.data, createdByUserId: session.user.id } })
}
```

## Context Providers

Les Context providers sont des Client Components — les isoler dans un fichier dédié :

```tsx
// app/providers.tsx
"use client"
import { QueryClient, QueryClientProvider } from "@tanstack/react-query"

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient())
  return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
}
```

## Server Components imbriqués dans Client Components

```tsx
// INTERDIT — casse les avantages Server Component
"use client"
function ClientWrapper() {
  return <ServerComponent />
}

// CORRECT — passer comme children
"use client"
function ClientWrapper({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>
}
// Dans le parent (Server) :
<ClientWrapper><ServerComponent /></ClientWrapper>
```

## URL State

- **Filtres, pagination, tri, recherche** → `nuqs` (pas de `useState`)
- **State UI local** (modal ouvert, sidebar) → `Zustand`
- **State serveur** (données API) → `TanStack Query`

```tsx
import { useQueryState } from "nuqs"

const [status, setStatus] = useQueryState("status", { defaultValue: "open" })
const [page, setPage] = useQueryState("page", { defaultValue: "1" })
```

## TanStack Query — Hydration Pattern (App Router)

```tsx
// app/tickets/page.tsx (Server Component)
import { HydrationBoundary, dehydrate, QueryClient } from "@tanstack/react-query"

export default async function TicketsPage() {
  const queryClient = new QueryClient()
  await queryClient.prefetchQuery({ queryKey: ["tickets"], queryFn: fetchTickets })
  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <TicketList />
    </HydrationBoundary>
  )
}
```

## Structure de Fichiers

```
app/
├── (auth)/           ← groupe route auth
├── (dashboard)/      ← groupe route dashboard équipe
│   ├── tickets/
│   │   ├── page.tsx          ← Server Component (listing)
│   │   ├── [id]/
│   │   │   └── page.tsx      ← Server Component (détail)
│   │   └── loading.tsx       ← Suspense fallback
├── (portal)/         ← groupe route portail client
├── actions/          ← Server Actions
├── api/              ← Route Handlers SEULEMENT si webhook/externe
components/
├── ui/               ← shadcn/ui (ne pas modifier)
├── tickets/          ← composants domaine tickets
├── reports/          ← composants domaine rapports
lib/
├── prisma.ts         ← singleton Prisma
├── auth.ts           ← Better-Auth config
├── email.ts          ← Resend config
├── r2.ts             ← Cloudflare R2 config
```

## Named Exports Obligatoires

```tsx
// INTERDIT
export default function TicketCard() {}

// CORRECT
export function TicketCard() {}
```
