---
description: Créer une branche + worktree pour travailler dessus isolément, puis ouvrir une PR avec gh. Usage: /git:worktree-start nom-de-la-branche
---

Workflow worktree — phase démarrage.

Le nom de branche demandé est : $ARGUMENTS

## Étapes à suivre strictement dans cet ordre

1. **Vérifier qu'on est sur la bonne branche de base**
   ```bash
   git branch --show-current
   git status
   ```
   Si des changements non committés existent, les gérer avant de continuer.

2. **Créer la branche depuis main**
   ```bash
   git branch $ARGUMENTS
   ```

3. **Créer le worktree**
   Le worktree doit être créé UN niveau au-dessus du repo (sibling), pas à l'intérieur.
   ```bash
   git worktree add ../Support_Manager-$ARGUMENTS $ARGUMENTS
   ```
   Vérifier avec `git worktree list` que ça a marché.

4. **Faire le travail dans le worktree**
   Tous les fichiers à modifier sont dans le chemin du worktree `../Support_Manager-$ARGUMENTS`.
   Utiliser les règles de `.claude/rules/` et les skills disponibles.

5. **Commit dans le worktree**
   ```bash
   cd ../Support_Manager-$ARGUMENTS
   git add <fichiers spécifiques, JAMAIS git add -A ou git add .>
   git commit -m "type(scope): description courte en impératif présent"
   ```
   **Règles commits :**
   - Format : `feat(tickets): add status history`, `fix(pdf): prevent empty report`
   - Auteur configuré : `James10192 <djedjelipatrick@gmail.com>`
   - JAMAIS "Generated with Claude Code" ni "Co-Authored-By" dans les commits
   - JAMAIS "Generated with Claude Code" dans le body de la PR

6. **Pusher la branche**
   ```bash
   git push -u origin $ARGUMENTS
   ```

7. **Créer une issue GitHub si nécessaire**
   Si la feature correspond à une issue à tracker :
   ```bash
   gh issue create \
     --title "feat: <description courte>" \
     --body "$(cat <<'EOF'
   ## Contexte
   <Pourquoi cette feature est nécessaire>

   ## Solution proposée
   <Ce qui va être implémenté>

   ## Critères d'acceptation
   - [ ] Critère 1
   - [ ] Critère 2
   EOF
   )" \
     --label "enhancement"
   ```
   Retourner le numéro d'issue créée.

8. **Créer la Pull Request avec gh**
   ```bash
   gh pr create \
     --title "type(scope): description courte" \
     --base main \
     --body "$(cat <<'EOF'
   ## Summary
   - <bullet 1 — changement principal>
   - <bullet 2 — si applicable>

   ## What was changed
   - `path/to/file.ts` — description du changement

   ## Test plan
   - [ ] Action à tester
   - [ ] Tests existants passent : pnpm test

   Closes #<numéro issue si applicable>
   EOF
   )"
   ```
   Afficher l'URL de la PR créée.

9. **Informer l'utilisateur**
   Après la PR créée, dire :
   "PR créée. Après review et merge, lancez `/git:worktree-finish $ARGUMENTS`"

## Règles importantes
- JAMAIS `git add .` ou `git add -A`
- JAMAIS "Generated with Claude Code" ou "Co-Authored-By" dans commits ou PR
- Auteur commits : `James10192 <djedjelipatrick@gmail.com>`
- Le worktree est un répertoire sibling : `../Support_Manager-<nom-branche>`
- Toujours créer la PR depuis la branche feature vers `main` (pas `presentation`)
- Si `gh` retourne une erreur auth → demander à l'utilisateur de faire `gh auth login`
