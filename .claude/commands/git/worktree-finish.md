---
description: Après merge d'une PR créée via worktree-start : vérifier la PR, supprimer le worktree, supprimer les branches, puller main à jour. Usage: /git:worktree-finish nom-de-la-branche
---

Workflow worktree — phase nettoyage après merge.

La branche mergée est : $ARGUMENTS

## Étapes à suivre strictement dans cet ordre

1. **Vérifier que la PR est bien mergée**
   ```bash
   gh pr view $ARGUMENTS --json state,mergedAt,number
   ```
   Si `state` n'est pas `MERGED` → **stopper et informer l'utilisateur** que la PR n'est pas encore mergée.

2. **Supprimer le worktree**
   D'abord récupérer le chemin exact avec `git worktree list`, puis supprimer :
   ```bash
   git worktree list
   git worktree remove ../Support_Manager-$ARGUMENTS --force
   ```
   Le `--force` est nécessaire parce que la branche locale n'a pas encore été mergée localement.

3. **Supprimer la branche locale**
   La branche n'est mergée que sur GitHub (merge commit côté remote), donc `git branch -d` va refuser. On utilise `-D` :
   ```bash
   git branch -D $ARGUMENTS
   ```

4. **Supprimer la branche distante**
   ```bash
   git push origin --delete $ARGUMENTS
   ```
   Si déjà supprimée automatiquement par GitHub → ignorer l'erreur.

5. **Puller main à jour**
   ```bash
   git checkout main
   git pull origin main
   ```
   Le pull va récupérer le merge commit de la PR.

6. **Fermer l'issue GitHub si applicable**
   Si une issue avait été créée avec worktree-start :
   ```bash
   gh issue close <numéro> --comment "Fermé via merge de la PR #<numéro PR>"
   ```
   (Vérifier avec `gh issue list --state open` si besoin)

7. **Vérifier**
   ```bash
   git worktree list   # doit montrer uniquement le repo principal
   git log --oneline -3  # doit montrer le merge de la PR en tête
   git branch -a | grep $ARGUMENTS  # ne doit rien retourner
   ```

## Règles importantes
- Toujours vérifier que la PR est MERGED avant de nettoyer
- Toujours utiliser `git worktree list` AVANT de supprimer pour confirmer le chemin
- Toujours utiliser `--force` sur `worktree remove`
- Toujours utiliser `-D` (majuscule) sur `branch` pour supprimer une branche non-mergée localement
- Toujours revenir sur `main` à la fin (pas `presentation`)
- Supprimer aussi la branche distante avec `git push origin --delete`
