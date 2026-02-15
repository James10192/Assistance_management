---
name: component
description: Créer un composant React correctement - Server vs Client, props typées, shadcn/ui patterns
---

# Skill : Créer un Composant

## Usage
`/component <nom du composant> [description de ce qu'il fait]`

---

## Phase 1 — Analyser le Besoin

Déterminer automatiquement :
1. **Server ou Client Component ?**
   - Nécessite `useState`, `useEffect`, events, browser APIs → **Client**
   - Affiche des données, pas d'interactivité → **Server**
   - Les deux → **Séparer en deux composants** (Server wrapper + Client island)

2. **Composants shadcn/ui à utiliser ?**
   - Formulaire → `Form` + `FormField` + `Input`/`Select`/`Textarea` + RHF
   - Tableau de données → `DataTable` (TanStack Table) + `DataTablePagination`
   - Dialogue/Modal → `Dialog` ou `AlertDialog`
   - Navigation → `Sidebar` + `SidebarContent`
   - Métriques → `Card` + `CardHeader` + `CardContent`
   - Graphiques → `ChartContainer` + composants Recharts

3. **Où placer le fichier ?**
   - Composant domaine → `components/<domain>/`
   - Composant UI générique → `components/ui/` (shadcn, ne pas modifier)
   - Composant page → `app/(dashboard)/<route>/components/`

---

## Phase 2 — Présenter le Plan

Montrer :
- Server ou Client et pourquoi
- Props interface TypeScript complète
- Composants shadcn/ui utilisés
- Si formulaire : schéma Zod exact + Server Action appelée

**Attendre OKAY.**

---

## Phase 3 — Créer le Composant

### Template Server Component

```tsx
// components/tickets/ticket-card.tsx
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Badge } from "@/components/ui/badge"
import type { TicketWithDetails } from "@/lib/types"

interface TicketCardProps {
  ticket: TicketWithDetails
}

export function TicketCard({ ticket }: TicketCardProps) {
  return (
    <Card>
      <CardHeader>
        <CardTitle>{ticket.ticketNumber}</CardTitle>
        <Badge variant={getPriorityVariant(ticket.priority)}>{ticket.priority}</Badge>
      </CardHeader>
      <CardContent>
        <p>{ticket.title}</p>
      </CardContent>
    </Card>
  )
}
```

### Template Client Component (Formulaire)

```tsx
// components/tickets/create-ticket-form.tsx
"use client"
import { useForm } from "react-hook-form"
import { zodResolver } from "@hookform/resolvers/zod"
import { z } from "zod"
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from "@/components/ui/form"
import { Input } from "@/components/ui/input"
import { Button } from "@/components/ui/button"
import { createTicket } from "@/app/actions/tickets"
import { toast } from "sonner"

const schema = z.object({
  title: z.string().min(1, "Titre requis").max(200),
})
type FormData = z.infer<typeof schema>

export function CreateTicketForm() {
  const form = useForm<FormData>({ resolver: zodResolver(schema) })

  async function onSubmit(data: FormData) {
    const formData = new FormData()
    Object.entries(data).forEach(([k, v]) => formData.append(k, v))
    const result = await createTicket(formData)
    if (result.success) toast.success("Ticket créé")
    else toast.error("Erreur lors de la création")
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)}>
        <FormField
          control={form.control}
          name="title"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Titre</FormLabel>
              <FormControl><Input {...field} /></FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit">Créer le ticket</Button>
      </form>
    </Form>
  )
}
```

### Template DataTable

```tsx
// components/tickets/tickets-data-table.tsx
"use client"
import { useReactTable, getCoreRowModel, ColumnDef } from "@tanstack/react-table"
import { DataTable } from "@/components/ui/data-table"
import { useQueryState } from "nuqs"
import type { TicketWithDetails } from "@/lib/types"

const columns: ColumnDef<TicketWithDetails>[] = [
  { accessorKey: "ticketNumber", header: "Référence" },
  { accessorKey: "title", header: "Titre" },
  { accessorKey: "priority", header: "Priorité" },
  { accessorKey: "status", header: "Statut" },
]

export function TicketsDataTable({ data }: { data: TicketWithDetails[] }) {
  const [page, setPage] = useQueryState("page", { defaultValue: "1" })
  const table = useReactTable({ data, columns, getCoreRowModel: getCoreRowModel() })
  return <DataTable table={table} />
}
```

---

## Checklist Composant

- [ ] Named export (pas `export default`)
- [ ] Interface Props typée explicitement
- [ ] Pas de `any`
- [ ] `"use client"` seulement si vraiment nécessaire
- [ ] Composants shadcn/ui existants utilisés (pas de HTML brut)
- [ ] Toast Sonner pour feedback utilisateur (pas `alert()`)
- [ ] Formulaires : RHF + Zod + shadcn Form
- [ ] URL state : `nuqs` (pas `useState` pour filtres/pagination)
