# Règles Prisma 6.x

## Singleton Obligatoire

Sans ce pattern, Next.js hot reload crée des centaines d'instances PrismaClient :

```typescript
// lib/prisma.ts
import { PrismaClient } from "@prisma/client"

const globalForPrisma = global as unknown as { prisma: PrismaClient }

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === "development" ? ["query", "error", "warn"] : ["error"],
  })

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma
```

## Migrations

```bash
# DEV UNIQUEMENT — génère le fichier + applique
npx prisma migrate dev --name add_ticket_history

# PRODUCTION UNIQUEMENT — applique les migrations en attente
npx prisma migrate deploy

# JAMAIS migrate dev en production
```

**Jamais modifier une migration déjà committée** — créer une nouvelle migration.

## Dual URL (Neon / Supabase / PgBouncer)

```prisma
// prisma/schema.prisma
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")   // URL poolée (PgBouncer/Neon)
  directUrl = env("DIRECT_URL")     // URL directe (utilisée par Prisma CLI)
}
```

Variables `.env` :
```
DATABASE_URL="postgresql://...?pgbouncer=true&connection_limit=1"
DIRECT_URL="postgresql://..."
```

## Seeding

```typescript
// prisma/seed.ts
import { prisma } from "../lib/prisma"

async function main() {
  await prisma.product.createMany({
    data: [
      { name: "KLASSCI College", code: "KLASS-C", isActive: true },
      { name: "KLASSCI University", code: "KLASS-U", isActive: true },
    ],
    skipDuplicates: true,
  })
}

main().catch(console.error).finally(() => prisma.$disconnect())
```

```json
// package.json
{ "prisma": { "seed": "tsx prisma/seed.ts" } }
```

## Types Natifs — Ne Pas Recréer

```typescript
// Utiliser les types Prisma générés, pas des types manuels
import type { Prisma } from "@prisma/client"

// Type pour les relations incluses
const ticketWithDetails = Prisma.validator<Prisma.TicketDefaultArgs>()({
  include: {
    product: true,
    client: true,
    assignedTo: { select: { id: true, name: true, email: true } },
    attachments: true,
    comments: { where: { isInternal: false } },
  },
})
export type TicketWithDetails = Prisma.TicketGetPayload<typeof ticketWithDetails>

// Type pour création
type CreateTicketInput = Prisma.TicketCreateInput
```

## Select — Ne Jamais Sur-Fetcher

```typescript
// INTERDIT — fetche tout
const user = await prisma.user.findUnique({ where: { id } })

// CORRECT — seulement ce qui est nécessaire
const user = await prisma.user.findUnique({
  where: { id },
  select: { id: true, name: true, email: true, role: true },
})
```

## Transactions pour Opérations Multi-Write

```typescript
// Toujours une transaction pour des écrits liés
const [ticket, history] = await prisma.$transaction([
  prisma.ticket.update({ where: { id }, data: { status: "resolved" } }),
  prisma.ticketHistory.create({
    data: { ticketId: id, fromStatus: "in_progress", toStatus: "resolved", changedByUserId: userId },
  }),
])
```

## Numérotation Atomique des Tickets

```typescript
// Génération séquentielle sans race condition
async function generateTicketNumber(productCode: string, type: string): Promise<string> {
  const year = new Date().getFullYear()
  const typeCode = { bug: "BUG", assistance: "INT", feature: "FEAT" }[type] ?? "TCK"

  const counter = await prisma.ticketCounter.upsert({
    where: { productCode_typeCode_year: { productCode, typeCode, year } },
    update: { lastSequence: { increment: 1 } },
    create: { productCode, typeCode, year, lastSequence: 1 },
    select: { lastSequence: true },
  })

  return `${productCode}-${typeCode}-${year}-${String(counter.lastSequence).padStart(4, "0")}`
}
```

## Schéma Support Manager — Tables Clés

```prisma
model TicketHistory {
  id             String   @id @default(cuid())
  ticketId       String
  ticket         Ticket   @relation(fields: [ticketId], references: [id])
  fromStatus     String
  toStatus       String
  changedByUserId String?
  changedBy      User?    @relation(fields: [changedByUserId], references: [id])
  note           String?
  createdAt      DateTime @default(now())
}

model ClientContact {
  id             String   @id @default(cuid())
  clientId       String
  client         Client   @relation(fields: [clientId], references: [id])
  name           String
  email          String   @unique
  // Auth géré par Better-Auth client portal
  createdAt      DateTime @default(now())
}

model TicketCounter {
  productCode String
  typeCode    String
  year        Int
  lastSequence Int    @default(0)
  @@unique([productCode, typeCode, year])
}
```
