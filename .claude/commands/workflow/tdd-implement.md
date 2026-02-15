# Workflow : Implémenter en TDD

Implémente une feature en suivant strictement le cycle RED → GREEN → REFACTOR.

## Usage

`/workflow:tdd-implement <description de la feature ou User Story>`

## Principe Fondamental

**JAMAIS écrire l'implémentation avant que le test échoue.** C'est la règle #1 du TDD. Si on ne voit pas le rouge, on ne sait pas si le test testait vraiment quelque chose.

---

## Phase 0 — Comprendre et Planifier

Avant d'écrire le premier test :
1. Lire les fichiers concernés (Server Actions, composants, schéma Prisma)
2. Identifier les cas à tester :
   - Le cas nominal (happy path)
   - Les cas d'erreur (input invalide, non autorisé, not found)
   - Les edge cases (valeurs limites, données manquantes)

Présenter la liste des tests prévus. **Attendre OKAY.**

---

## Phase 1 — RED : Écrire le Test qui Échoue

Écrire UN test, le plus simple et le plus précis possible :

```typescript
// tests/unit/actions/tickets.test.ts
import { createTicket } from "@/app/actions/tickets"

describe("createTicket", () => {
  it("retourne une erreur quand le titre est vide", async () => {
    const formData = new FormData()
    // titre manquant

    const result = await createTicket(formData)

    expect(result.success).toBe(false)
    expect(result.error).toBeDefined()
  })
})
```

**Exécuter le test — vérifier qu'il est RED :**
```bash
pnpm test tests/unit/actions/tickets.test.ts
```

Montrer la sortie du test échoué. **Stopper. Attendre confirmation que le rouge est vu.**

---

## Phase 2 — GREEN : Implémentation Minimale

Écrire **seulement** ce qui est nécessaire pour que le test passe.
Pas d'optimisation. Pas d'abstraction. Pas de "au cas où".

```typescript
// app/actions/tickets.ts
"use server"
import { z } from "zod"

const CreateTicketSchema = z.object({
  title: z.string().min(1, "Titre requis"),
})

export async function createTicket(formData: FormData) {
  const parsed = CreateTicketSchema.safeParse(Object.fromEntries(formData))
  if (!parsed.success) return { success: false, error: parsed.error.flatten() }

  // ... implémentation minimale
  return { success: true, data: ticket }
}
```

**Exécuter — vérifier que c'est GREEN :**
```bash
pnpm test tests/unit/actions/tickets.test.ts
```

Montrer la sortie verte. **Confirmer que le test passe.**

---

## Phase 3 — Ajouter le Prochain Test

Répéter le cycle RED → GREEN pour chaque cas de test planifié en Phase 0.

Ne pas passer au cas suivant avant que le cas courant soit GREEN et que les tests précédents passent encore.

```bash
# Vérifier que rien n'est cassé
pnpm test
```

---

## Phase 4 — REFACTOR (seulement si clairement nécessaire)

Une fois tous les tests GREEN :
- Nettoyer le code **sans changer le comportement**
- Extraire une répétition évidente (3+ occurrences identiques)
- Améliorer la lisibilité si confuse

**JAMAIS créer des abstractions** pour une utilisation unique.
**JAMAIS modifier les tests** pendant le refactoring (sauf si le test était mal écrit).

```bash
# Les tests doivent rester GREEN après refactoring
pnpm test
```

---

## Phase 5 — Commit

```bash
# Ajouter explicitement les fichiers
git add app/actions/tickets.ts tests/unit/actions/tickets.test.ts

# Commit avec le bon format (sans Co-Authored-By)
git commit -m "feat(tickets): add createTicket server action with Zod validation"
```

Auteur configuré : `James10192 <djedjelipatrick@gmail.com>`

---

## Règles TDD

- **Un seul test à la fois** — pas de batch de 5 tests d'un coup
- **Voir le rouge avant d'écrire le code** — obligatoire
- **Voir le vert avant de passer au test suivant** — obligatoire
- **Ne pas modifier l'implémentation pendant le refactoring** — uniquement la structure
- **Si un test est difficile à écrire** → c'est souvent signe que le design est mauvais, revoir l'architecture
- **Tests de Server Actions** → tester le comportement, pas l'implémentation interne
