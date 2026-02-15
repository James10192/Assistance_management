# Workflow : Code Review

Review structurée d'un diff ou d'une Pull Request avant merge.

## Usage

- `/workflow:code-review` — review du diff `main...HEAD` courant
- `/workflow:code-review <PR_NUMBER>` — review d'une PR GitHub spécifique

## Étapes

### 1. Collecter le diff

Si numéro de PR fourni :
```bash
gh pr diff $ARGUMENTS
gh pr view $ARGUMENTS --json title,body,files
```

Sinon :
```bash
git diff main...HEAD
git log main...HEAD --oneline
```

### 2. Analyse Structurée

Analyser selon ces 6 axes **dans cet ordre** :

#### A. Correctness (Fonctionnement)
- Le code fait-il ce qu'il est censé faire ?
- Y a-t-il des edge cases non gérés ?
- Les conditions aux limites sont-elles correctes ?
- Les types de retour sont-ils cohérents ?

#### B. Sécurité
- Les inputs serveur sont-ils validés avec Zod ?
- L'authentification est-elle vérifiée dans chaque Server Action ?
- L'autorisation RBAC est-elle vérifiée (bon rôle) ?
- Des secrets dans le code ? (`process.env` utilisé partout ?)
- Risques XSS, injection SQL, path traversal ?

#### C. Performance
- Des requêtes N+1 ? (boucle avec `findUnique` à l'intérieur)
- Sur-fetching ? (colonnes inutiles sélectionnées)
- `useEffect` pour fetcher des données ? (devrait être Server Component)
- Pas de pagination sur des listes potentiellement grandes ?

#### D. Architecture Next.js 15
- `"use client"` justifié ?
- `params`/`searchParams` awaités ?
- Appel de ses propres Route Handlers depuis Server Component ?
- URL state géré avec `nuqs` ?
- `export default` sur un composant non-page ?

#### E. TypeScript
- Pas de `any` ?
- Pas de `as` forcé sans type guard ?
- Interfaces Props correctement typées ?
- Types Prisma natifs utilisés (`Prisma.TicketGetPayload`) ?

#### F. Maintenabilité
- Code trop abstrait pour 1-2 usages ?
- Nommage clair et cohérent avec les conventions du projet ?
- Fonctions trop longues (>50 lignes) ?
- Commentaires ajoutés sur du code non modifié ? (interdit)

### 3. Format du Rapport

```markdown
## Code Review — <nom de la branche ou PR #N>

### Résumé
<1-3 phrases sur l'ensemble du changement>

### Issues Critiques 🔴 (bloqueront la PR)
- `src/app/actions/tickets.ts:45` — Pas de validation Zod sur `clientId`, risque d'injection

### Issues Importantes 🟡 (à corriger avant merge)
- `components/tickets/list.tsx:12` — `useEffect` pour fetcher, devrait être Server Component

### Suggestions 🟢 (optionnel, amélioration)
- `lib/types.ts:8` — Le type `TicketData` duplique `Prisma.TicketGetPayload`, utiliser directement le type Prisma

### Points Positifs ✅
- Validation Zod complète dans createTicket
- Types correctement inférés depuis Prisma
```

### 4. Action Recommandée

Conclure avec :
- **APPROUVER** — aucune issue critique ou importante
- **RÉVISIONS REQUISES** — issues 🔴 ou 🟡 à corriger
- **REFACTORISATION** — problèmes architecturaux profonds

### 5. Optionnel : Poster en Commentaire de PR

Si un numéro de PR est fourni et que l'utilisateur le demande :
```bash
gh pr review $ARGUMENTS --comment --body "<rapport>"
```

---

## Règles

- Référencer toujours les problèmes avec `fichier:ligne`
- Pas de suggestions de refactoring pour du code hors scope du diff
- Distinguer clairement bloquant vs suggestion
- Pas de "Generated with Claude Code" dans les commentaires postés
