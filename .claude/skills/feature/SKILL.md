---
name: feature
description: Workflow complet pour implémenter une nouvelle feature - de la compréhension à l'implémentation TDD
---

# Skill : Implémenter une Feature

## Usage
`/feature <description de la feature ou référence à la section PRD>`

## Ce que ce skill fait

Orchestre le workflow complet : comprendre → planifier → implémenter en TDD → valider.

---

## Phase 1 — Comprendre (lecture seule)

1. Lire la description de la feature passée en argument
2. Si une référence PRD est mentionnée, lire `PRD_Support_Manager_ADC.md` pour la section concernée
3. Identifier les fichiers impactés : modèle Prisma, Server Actions, composants, routes
4. Identifier les entités domaine impliquées et leurs relations
5. Identifier les contraintes : auth, RBAC, validations, règles métier

**Aucune modification à cette phase.**

---

## Phase 2 — Présenter le Plan

Présenter sous cette structure :

### Entités & Schéma Prisma
- Quels modèles sont modifiés ou créés ?
- Nouvelles colonnes/relations ?
- Migration nécessaire ?

### Server Actions / API
- Quelles actions créer ou modifier ?
- Validation Zod : schéma exact
- Permissions vérifiées

### Composants UI
- Server Components vs Client Components (justification)
- Quels composants shadcn/ui utiliser
- Structure des props (interfaces TypeScript)

### Plan d'Implémentation (ordonné par dépendance)
1. Migration Prisma
2. Types & validation
3. Server Actions
4. Composants
5. Tests

**Attendre OKAY avant de continuer.**

---

## Phase 3 — Implémenter en TDD

Suivre le cycle RED → GREEN → REFACTOR pour chaque unité :

### RED : Écrire le test en premier
```typescript
// tests/tickets/create-ticket.test.ts
describe("createTicket", () => {
  it("should return error when title is empty", async () => {
    const result = await createTicket(new FormData()) // formData vide
    expect(result.success).toBe(false)
  })
})
```
Stopper, montrer le test. Attendre confirmation avant d'écrire l'implémentation.

### GREEN : Implémentation minimale pour passer le test
Écrire seulement ce qui est nécessaire pour faire passer le test.
Montrer les résultats de test.

### REFACTOR : Améliorer sans casser
Nettoyer uniquement si clairement nécessaire. Pas d'abstraction prématurée.

---

## Phase 4 — Checklist Finale

Avant de déclarer la feature terminée :

- [ ] Tests passent
- [ ] Validation Zod sur tous les inputs serveur
- [ ] Auth vérifiée dans chaque Server Action
- [ ] RBAC vérifié (l'utilisateur a-t-il le bon rôle ?)
- [ ] Pas de `any` TypeScript
- [ ] Pas de `"use client"` inutile
- [ ] `params` et `searchParams` awaités (Next.js 15)
- [ ] Commits atomiques avec format correct (sans Co-Authored-By)
- [ ] Auteur : `James10192 <djedjelipatrick@gmail.com>`

---

## Règle Si Découverte en Cours d'Implémentation

Si on découvre quelque chose qui change le plan → **stopper, re-présenter la modification, attendre OKAY** avant de continuer.
