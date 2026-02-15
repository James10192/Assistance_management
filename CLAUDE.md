# Support Manager — Claude Code Memory

## Projet

**Support Manager** — plateforme B2B centralisée de gestion de support technique et communication client pour ADC.
**Produits gérés :** KLASSCI College, KLASSCI University (et futurs produits)
**Owner :** Marcel Djedje-li
**Spec détaillée :** @PRD_Support_Manager_ADC.md

---

## Stack Technique (TRANCHÉE — ne pas dévier)

```
Framework :      Next.js 15 (App Router, TypeScript strict)
Base de données: PostgreSQL
ORM :            Prisma 6.x
Auth :           Better-Auth + plugins organization + teams
Forms :          React Hook Form v7 + Zod v4
State serveur :  TanStack Query v5
State client :   Zustand v5
URL state :      nuqs
UI :             shadcn/ui + Tailwind CSS v4
Email :          Resend + React Email
Fichiers :       Cloudflare R2 + Presigned URLs
PDF :            @react-pdf/renderer
Toasts :         Sonner
Tables :         TanStack Table v8 (via shadcn DataTable)
Package manager: pnpm
```

---

## Commandes Clés

```bash
pnpm dev                          # Démarrer le serveur de développement
pnpm build && pnpm start          # Build production
pnpm lint                         # ESLint
pnpm format                       # Prettier
pnpm test                         # Tests
npx prisma migrate dev --name X   # Nouvelle migration (dev seulement)
npx prisma migrate deploy         # Appliquer migrations (prod)
npx prisma db seed                # Seeder la base
npx prisma studio                 # Interface DB visuelle
```

---

## Règles Absolues (JAMAIS violer)

1. **JAMAIS coder sans OKAY** — utiliser `/workflow:plan-and-confirm` pour tout changement non trivial
2. **JAMAIS `git add -A` ou `git add .`** — toujours ajouter les fichiers explicitement par nom
3. **JAMAIS commiter** sans demande explicite de l'utilisateur
4. **JAMAIS créer de README, docs .md** sauf si explicitement demandé
5. **JAMAIS force-push sur main**
6. **JAMAIS modifier une migration déjà committée** — créer une nouvelle migration
7. **JAMAIS utiliser `any` en TypeScript** — typer correctement ou utiliser `unknown`
8. **JAMAIS appeler ses propres Route Handlers depuis un Server Component** — appeler la fonction directement
9. **Si une erreur se répète 2 fois** — stopper, signaler et demander une reformulation du prompt plutôt que réessayer

---

## Règles Architecture Next.js 15

- **Server Components par défaut** — `"use client"` SEULEMENT si hooks, events, browser APIs, ou state
- **`params` et `searchParams` sont des Promises** → toujours les `await` en Next.js 15
- **Server Actions** → validation Zod obligatoire avec `safeParse()` avant toute opération DB
- **Data fetching** → `async/await` dans Server Components, jamais `useEffect` pour fetcher
- **Context providers** → Client Components séparés dans `providers.tsx`
- **URL state** → utiliser `nuqs`, pas de `useState` pour filtres/pagination/tri
- **Named exports** uniquement — pas de `export default` pour les composants

---

## Règles Prisma

- **Singleton obligatoire** dans `lib/prisma.ts` (pattern `globalForPrisma`)
- **`migrate dev`** → dev uniquement. **`migrate deploy`** → production
- **Dual URL** obligatoire pour Neon/PgBouncer (`DATABASE_URL` + `DIRECT_URL`)
- **Types Prisma natifs** → utiliser `Prisma.TicketGetPayload<>` plutôt que recréer des types
- **`connection_limit=1`** dans l'URL pour serverless/Vercel

---

## Domaine Métier — Entités Clés

```
Product           → has many Tickets, ClientProducts, ProductVersions
ProductVersion    → belongs to Product, has many Tickets
Client            → has many ClientContacts, ClientProducts
ClientContact     → auth séparée (Better-Auth client portal)
Ticket            → types: bug | assistance | feature | release
                    statuts: open → assigned → in_progress → resolved → validated → closed
                    belongs to Product, ProductVersion, Client
TicketHistory     → audit log de chaque transition de statut (QUI, QUOI, QUAND)
TicketComment     → is_internal: true = visible équipe seulement
TicketAttachment  → uploaded_by peut être un User OU un ClientContact
AssistanceReport  → PDF généré après résolution d'un ticket assistance
Release           → groupe de features, workflow validation manager avant envoi
EmailLog          → tracking ouverture/clics via Resend webhooks
```

**Numérotation tickets :** `{PRODUCT_CODE}-{TYPE_CODE}-{YYYY}-{SEQ:04d}`
Format exemple : `KLASS-BUG-2026-0001` — séquence atomique par produit/année en DB

---

## Phases du Projet

- **Phase 1 MVP :** Tickets (bug + assistance) + PDF rapports + auth interne + rôles Admin/Developer
- **Phase 2 :** Templates email + workflow validation Manager + notifications auto + multi-produits
- **Phase 3 :** Release Notes éditeur + widget embeddable + portail client basique
- **Phase 4 :** Analytics avancées + portail client complet + API publique

---

## Anti-Patterns — Ne PAS faire

- Pas de `useEffect` pour fetcher des données
- Pas de helpers/utils pour usage unique — 3 lignes similaires valent mieux qu'une abstraction prématurée
- Pas de docstrings sur du code non modifié
- Pas de gestion d'erreurs pour des scénarios impossibles
- Pas de feature flags ou shims de rétrocompatibilité sauf si demandé
- Pas de nouveaux packages sans demande explicite
- Pas de Route Handler si une Server Action ou appel direct suffit

---

## Commandes Disponibles

| Commande | Usage |
|----------|-------|
| `/workflow:plan-and-confirm` | OBLIGATOIRE pour tout changement non trivial |
| `/workflow:create-pr` | Créer une Pull Request avec `gh pr create` |
| `/workflow:code-review` | Review structurée d'un diff ou PR |
| `/workflow:tdd-implement` | Implémenter en TDD (RED → GREEN → REFACTOR) |
| `/git:worktree-start <branch>` | Démarrer une feature en isolation |
| `/git:worktree-finish <branch>` | Nettoyer après merge |
| `/refactor:refactor-code` | Analyse avant de toucher du code legacy |
| `/security:security-audit` | Audit OWASP avant release |
| `/anthropic:update-memory-bank` | Mettre à jour CLAUDE.md après session |

---

## Fichiers Mémoire Détaillés

@.claude/rules/nextjs.md      — règles App Router détaillées
@.claude/rules/prisma.md      — patterns Prisma et migrations
@.claude/rules/typescript.md  — conventions TypeScript strictes
@.claude/rules/workflow.md    — workflow de développement

*Document vivant — mettre à jour après chaque décision architecturale majeure*
