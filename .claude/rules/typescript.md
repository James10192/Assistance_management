# Règles TypeScript Strict

## Configuration de Base

```json
// tsconfig.json — strict mode obligatoire
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

## Interdictions Absolues

```typescript
// INTERDIT — ne jamais utiliser any
const data: any = response.json()
function process(input: any) {}

// INTERDIT — as forcé sans vérification
const user = data as User

// CORRECT — typer proprement
const data: unknown = response.json()
if (isUser(data)) { /* data est User ici */ }

// CORRECT — type guard
function isUser(value: unknown): value is User {
  return typeof value === "object" && value !== null && "id" in value
}
```

## Exports

```typescript
// INTERDIT — default export pour composants/fonctions
export default function TicketCard() {}
export default async function createTicket() {}

// CORRECT — named exports
export function TicketCard() {}
export async function createTicket() {}

// Exception acceptée : default export pour pages Next.js
// (requis par la convention App Router)
export default function Page() {}
```

## Conventions de Nommage

| Élément | Convention | Exemple |
|---------|-----------|---------|
| Composants React | PascalCase | `TicketCard`, `AssistanceReport` |
| Fonctions/variables | camelCase | `createTicket`, `ticketId` |
| Constantes globales | SCREAMING_SNAKE | `MAX_FILE_SIZE`, `TICKET_STATUSES` |
| Types/Interfaces | PascalCase | `TicketWithDetails`, `CreateTicketInput` |
| Fichiers composants | kebab-case | `ticket-card.tsx`, `data-table.tsx` |
| Fichiers utilitaires | kebab-case | `format-date.ts`, `generate-pdf.ts` |
| Dossiers | kebab-case | `ticket-details/`, `assistance-reports/` |
| Enums (valeurs) | snake_case | `ticket.status === "in_progress"` |

## Interfaces vs Types

```typescript
// Préférer type pour les unions et intersections
type TicketStatus = "open" | "assigned" | "in_progress" | "resolved" | "validated" | "closed"
type TicketType = "bug" | "assistance" | "feature" | "release"
type Priority = "low" | "medium" | "high" | "critical"

// Préférer interface pour les objets extensibles
interface TicketFilters {
  status?: TicketStatus
  type?: TicketType
  priority?: Priority
  productId?: string
  clientId?: string
  assignedToId?: string
  page?: number
  pageSize?: number
}
```

## Props de Composants

```typescript
// CORRECT — typer explicitement les props
interface TicketCardProps {
  ticket: TicketWithDetails
  onStatusChange?: (newStatus: TicketStatus) => void
  showActions?: boolean
}

export function TicketCard({ ticket, onStatusChange, showActions = true }: TicketCardProps) {}
```

## Zod v4 — Patterns

```typescript
import { z } from "zod"

// Utiliser les fonctions top-level Zod v4 (tree-shakable)
const schema = z.object({
  email: z.email(),           // pas z.string().email()
  website: z.url().optional(), // pas z.string().url()
  ticketId: z.cuid(),
})

// Inférer le type depuis le schéma
type CreateTicketForm = z.infer<typeof CreateTicketSchema>

// safeParse pour Server Actions (jamais parse directement)
const result = schema.safeParse(data)
if (!result.success) {
  return { error: result.error.flatten() }
}
```

## Gestion d'Erreurs Typée

```typescript
// Type de retour explicite pour Server Actions
type ActionResult<T> =
  | { success: true; data: T }
  | { success: false; error: string | z.ZodError["flatten"] }

export async function createTicket(formData: FormData): Promise<ActionResult<Ticket>> {
  const parsed = CreateTicketSchema.safeParse(Object.fromEntries(formData))
  if (!parsed.success) return { success: false, error: parsed.error.flatten() }

  try {
    const ticket = await prisma.ticket.create({ data: parsed.data })
    return { success: true, data: ticket }
  } catch {
    return { success: false, error: "Erreur lors de la création du ticket" }
  }
}
```

## Pas de Magic Strings

```typescript
// INTERDIT
if (ticket.status === "in_progress") {}
if (user.role === "admin") {}

// CORRECT — constantes typées
const TICKET_STATUSES = ["open", "assigned", "in_progress", "resolved", "validated", "closed"] as const
type TicketStatus = typeof TICKET_STATUSES[number]

const USER_ROLES = ["admin", "manager", "developer", "support"] as const
type UserRole = typeof USER_ROLES[number]
```
