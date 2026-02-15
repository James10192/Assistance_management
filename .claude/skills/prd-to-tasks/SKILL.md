---
name: prd-to-tasks
description: Décompose une section du PRD en tâches concrètes ordonnées par dépendance, prêtes à implémenter
---

# Skill : PRD → Tâches Concrètes

## Usage
`/prd-to-tasks <section ou feature du PRD>`

Exemples :
- `/prd-to-tasks Section 3.2 - Système de Tickets`
- `/prd-to-tasks gestion des rapports d'assistance PDF`
- `/prd-to-tasks Phase 1 MVP complet`

---

## Ce que ce skill fait

Lit le PRD (`PRD_Support_Manager_ADC.md`), analyse la section demandée, et produit une liste de tâches concrètes, ordonnées par dépendance technique, chacune faisable en une session Claude Code.

---

## Processus

### 1. Lire le PRD

Lire la section concernée de `PRD_Support_Manager_ADC.md`.
Identifier :
- Les entités de données impliquées
- Les règles métier (validations, workflows, permissions)
- Les interfaces utilisateur décrites
- Les contraintes non fonctionnelles

### 2. Décomposer en tâches

Format de chaque tâche :

```
### TASK-XXX : <titre court>
**Dépend de :** TASK-YYY (ou "aucune")
**Fichiers impactés :**
- `prisma/schema.prisma` — ajouter modèle X
- `app/actions/tickets.ts` — function createTicket()
- `components/tickets/create-ticket-form.tsx` — nouveau composant

**Ce que ça fait :** <1-2 phrases max>

**Critères de succès :**
- [ ] Migration Prisma crée sans erreur
- [ ] Server Action valide les inputs avec Zod
- [ ] Composant s'affiche avec les bonnes props
- [ ] Test E2E : créer un ticket depuis l'UI → apparaît dans la liste
```

### 3. Ordonner par dépendance

Toujours dans cet ordre :
1. **Schéma Prisma + Migration** (fondation)
2. **Types TypeScript + Schémas Zod** (contrats)
3. **Server Actions** (logique métier)
4. **Server Components** (pages, listing)
5. **Client Components** (formulaires, interactions)
6. **Tests** (validation)

### 4. Estimer la complexité

Pour chaque tâche :
- 🟢 **Simple** : 1 fichier, < 50 lignes — 15-30 min
- 🟡 **Moyen** : 2-4 fichiers, logique métier — 30-60 min
- 🔴 **Complexe** : 5+ fichiers, intégrations externes — décomposer davantage

Si une tâche est 🔴 : la décomposer en sous-tâches 🟢 ou 🟡.

---

## Exemple de Sortie

Pour `/prd-to-tasks Section 3.2 - Système de Tickets Unifié` :

```
## Tâches : Système de Tickets

### TASK-001 : Schéma Prisma — Modèles Ticket + TicketHistory + TicketCounter
🟡 Dépend de : aucune
Fichiers : prisma/schema.prisma, prisma/migrations/xxx
Critères : migrate dev sans erreur, types générés

### TASK-002 : Types & Schémas Zod pour Tickets
🟢 Dépend de : TASK-001
Fichiers : lib/types.ts, lib/validations/ticket.ts
Critères : types Prisma exportés, schémas Zod testés

### TASK-003 : Server Actions — CRUD Tickets
🟡 Dépend de : TASK-002
Fichiers : app/actions/tickets.ts
Critères : createTicket, updateTicketStatus, assignTicket — validés + auth

### TASK-004 : Page Listing Tickets (Server Component)
🟢 Dépend de : TASK-003
Fichiers : app/(dashboard)/tickets/page.tsx
Critères : affiche liste paginée, filtres URL via nuqs

### TASK-005 : DataTable Tickets (Client Component)
🟡 Dépend de : TASK-004
Fichiers : components/tickets/tickets-data-table.tsx
Critères : colonnes typées, tri, pagination, actions par ligne

### TASK-006 : Formulaire Création Ticket
🟡 Dépend de : TASK-003
Fichiers : components/tickets/create-ticket-form.tsx
Critères : RHF + Zod, feedback Sonner, redirect après succès

### TASK-007 : Page Détail Ticket
🟡 Dépend de : TASK-005 + TASK-006
Fichiers : app/(dashboard)/tickets/[id]/page.tsx
Critères : affiche tous les champs, boutons d'action selon rôle et statut
```

---

## Règle : Pas de Tâches Trop Grandes

Si une tâche touche plus de 6 fichiers ou contient plusieurs logiques métier distinctes → la couper en deux tâches indépendantes.
