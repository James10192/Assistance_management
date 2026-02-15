# Workflow : Créer une Pull Request

Crée une Pull Request GitHub complète et professionnelle à partir des changements de la branche courante.

## Étapes

### 1. Collecter le contexte (en parallèle)

```bash
git status
git diff main...HEAD
git log main...HEAD --oneline
git branch --show-current
```

### 2. Analyser TOUS les commits (pas seulement le dernier)

Lire l'intégralité du diff `main...HEAD`. Comprendre :
- Quels fichiers ont changé et pourquoi
- Quel problème cette PR résout
- Quels tests ont été ajoutés ou modifiés

### 3. Rédiger le titre et le corps

**Titre :** ≤ 70 caractères, format `type(scope): description` en impératif présent.
Exemple : `feat(tickets): add status transition history and audit log`

**Corps :** Utiliser exactement ce format :

```markdown
## Summary
- <bullet point 1 — changement principal>
- <bullet point 2 — changement secondaire si applicable>
- <bullet point 3 — si applicable>

## What was changed
- `path/to/file.ts` — description de ce qui a changé
- `prisma/schema.prisma` — ajout de X modèles

## Test plan
- [ ] <action à tester manuellement ou test automatisé>
- [ ] <vérification supplémentaire>
- [ ] Tests existants passent : `pnpm test`
```

### 4. Vérifications pré-PR

Avant de créer la PR :
- Vérifier qu'on n'est PAS sur `main` ou `master`
- Vérifier que la branche est poussée sur le remote
- Ne JAMAIS force-push sur main — avertir si demandé

```bash
# Pousser si nécessaire
git push -u origin $(git branch --show-current)
```

### 5. Créer la PR

```bash
gh pr create --title "<titre>" --body "$(cat <<'EOF'
## Summary
- <bullet 1>
- <bullet 2>

## What was changed
- `file.ts` — description

## Test plan
- [ ] Action 1
- [ ] Tests passent : pnpm test
EOF
)"
```

### 6. Retourner l'URL

Afficher l'URL de la PR créée pour que l'utilisateur puisse la consulter.

---

## Règles Absolues

- **PAS de "Generated with Claude Code"** dans le corps
- **PAS de "Co-Authored-By"** dans aucun commit ou description
- **JAMAIS force-push sur main/master** — refuser et avertir
- **JAMAIS créer une PR si des tests échouent** — signaler d'abord
- Titre en anglais, corps en français ou anglais selon la langue du projet
- Si la branche n'existe pas sur le remote → pousser d'abord, puis créer la PR
