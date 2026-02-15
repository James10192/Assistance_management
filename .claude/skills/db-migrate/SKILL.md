---
name: db-migrate
description: Workflow sécurisé pour créer et appliquer des migrations Prisma
---

# Skill : Migration Prisma Sécurisée

## Usage
`/db-migrate <description de la migration>`

---

## Phase 1 — Lire Avant Toucher

1. Lire `prisma/schema.prisma` en entier
2. Vérifier les migrations existantes dans `prisma/migrations/`
3. Identifier les dépendances de la migration demandée (FK, enums, etc.)

---

## Phase 2 — Présenter le Plan

Montrer exactement :
- Les modèles Prisma à créer ou modifier
- Les champs ajoutés avec leurs types, contraintes, valeurs par défaut
- Les relations (FK) avec le comportement `onDelete`
- Si des données existantes pourraient être impactées (breaking change)
- La commande exacte qui sera exécutée

**Attendre OKAY.**

---

## Phase 3 — Exécution

```bash
# 1. Vérifier l'état actuel
npx prisma migrate status

# 2. Créer la migration (DEV uniquement)
npx prisma migrate dev --name <description-kebab-case>

# 3. Vérifier le fichier généré
# Lire prisma/migrations/<timestamp>_<nom>/migration.sql

# 4. Mettre à jour le seed si nécessaire
# prisma/seed.ts

# 5. Régénérer le client
npx prisma generate
```

---

## Règles Impératives

- **JAMAIS modifier une migration déjà committée** — créer une nouvelle
- **JAMAIS `migrate dev` en production** — uniquement `migrate deploy`
- **Toujours** inspecter le SQL généré avant de le confirmer
- **Toujours** mettre à jour `prisma/seed.ts` si de nouvelles données de référence sont nécessaires
- **Toujours** utiliser `skipDuplicates: true` dans le seed pour idempotence

---

## Modèles Obligatoires pour Support Manager

Vérifier que ces modèles existent lors de toute migration :

```prisma
model TicketHistory {
  // Audit log OBLIGATOIRE — toute transition de statut doit être loggée
  id              String   @id @default(cuid())
  ticketId        String
  fromStatus      String
  toStatus        String
  changedByUserId String?
  note            String?
  createdAt       DateTime @default(now())
}

model TicketCounter {
  // Numérotation atomique sans race condition
  productCode  String
  typeCode     String
  year         Int
  lastSequence Int    @default(0)
  @@unique([productCode, typeCode, year])
}

model ClientContact {
  // Auth séparée pour le portail client — NE PAS mélanger avec Users
  id       String @id @default(cuid())
  clientId String
  email    String @unique
  name     String
}
```

---

## Patterns à Éviter

```prisma
// INTERDIT — pas de cascade delete sur des données critiques
clientId String
client   Client @relation(fields: [clientId], references: [id], onDelete: Cascade)

// CORRECT pour tickets (garder l'historique)
clientId String
client   Client @relation(fields: [clientId], references: [id], onDelete: Restrict)

// CORRECT pour pièces jointes (peut être supprimé avec le ticket)
ticketId String
ticket   Ticket @relation(fields: [ticketId], references: [id], onDelete: Cascade)
```
