# Règles Workflow de Développement

## Règle Fondamentale : Plan Before Code

**Jamais de code sans que le plan ait été approuvé.** Utiliser `/workflow:plan-and-confirm` pour tout changement non trivial. Un changement trivial = modifier une chaîne de caractères ou corriger une typo.

## Anti-Over-Engineering (Le Plus Violé par l'IA)

- **3 lignes similaires valent mieux qu'une abstraction prématurée**
- **Ne pas créer d'helpers/utils pour un usage unique**
- **Ne pas ajouter de gestion d'erreurs pour des scénarios impossibles**
- **Ne pas concevoir pour des besoins hypothétiques futurs**
- **Préférer modifier un fichier existant plutôt que créer un nouveau**
- **Pas de feature flags ou shims rétrocompatibles** sauf si demandé explicitement

```typescript
// INTERDIT — abstraction inutile pour 2 usages
function formatTicketBadgeLabel(type: string, status: string): string {
  return `${type}-${status}`
}

// CORRECT — inline si utilisé 1-2 fois
const label = `${ticket.type}-${ticket.status}`
```

## Ce Qu'on Ne Touche PAS Sans Demande

- Ne pas ajouter de docstrings/JSDoc sur du code non modifié
- Ne pas ajouter de commentaires évidents
- Ne pas refactorer le code adjacent au bug fixé
- Ne pas renommer des variables "pour la clarté" sans demande
- Ne pas convertir des `var` en `const` qui n'ont pas de rapport avec la tâche

## Format des Commits Git

```
<type>(<scope>): <description courte en impératif présent>

[corps optionnel — explication du POURQUOI si nécessaire]
```

**Types :** `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `style`

**Règles :**
- Description ≤ 72 caractères
- Impératif présent : "add ticket history" pas "added ticket history"
- PAS de "Generated with Claude Code" ou "Co-Authored-By" dans les commits
- Auteur des commits : `James10192 <djedjelipatrick@gmail.com>`

**Exemples corrects :**
```
feat(tickets): add ticket history audit log on status transition
fix(pdf): prevent empty report generation when resolution is missing
refactor(auth): extract client portal guard to separate middleware
```

## Git — Opérations Sûres

```bash
# Toujours ajouter explicitement (jamais git add -A ou git add .)
git add src/app/tickets/actions.ts prisma/migrations/xxx.sql

# Vérifier avant commit
git diff --staged

# Commit avec auteur configuré
git commit -m "feat(tickets): add status transition history"

# Créer une issue GitHub
gh issue create --title "..." --body "..." --label bug

# Créer une PR après review du diff
gh pr create --title "..." --body "..."

# Merger une PR après approbation
gh pr merge <number> --squash --delete-branch
```

## Worktree Workflow (Développement Parallèle)

```bash
# Démarrer une feature isolée
git worktree add ../support-manager-feature-tickets feature/tickets
cd ../support-manager-feature-tickets

# Travailler en isolation totale
# (commits, tests, sans affecter main)

# Après merge de la PR
git worktree remove ../support-manager-feature-tickets
git branch -d feature/tickets
```

## Séquence Complète Feature → Production

```
1. /git:worktree-start feature/nom-feature
2. /workflow:plan-and-confirm  ← OBLIGATOIRE
3. Coder (un fichier à la fois, commits atomiques)
4. /workflow:code-review       ← vérifier avant PR
5. /workflow:create-pr         ← créer la PR
6. Merge après approbation
7. /git:worktree-finish feature/nom-feature
8. /anthropic:update-memory-bank ← si décisions architecturales importantes
```

## Sécurité — Règles Obligatoires

- **Toujours valider les inputs côté serveur** avec Zod avant toute opération DB
- **Vérifier l'authentification** dans chaque Server Action
- **Vérifier l'autorisation** (RBAC) : un developer ne peut voir que les tickets de ses produits assignés
- **Jamais de secrets dans le code** — toujours `process.env.XXX`
- **Rate limiting** sur toutes les routes publiques (widget client, endpoint signature)
- **Sanitiser les inputs** avant génération PDF pour éviter injection

## Règle des Deux Corrections

Si Claude fait la même erreur **deux fois de suite** dans la même session :
1. Stopper immédiatement
2. Signaler : "J'ai fait cette erreur deux fois. Plutôt que de réessayer, reformulez le prompt en incluant la contrainte manquante."
3. Attendre la reformulation — ne pas essayer une troisième fois
