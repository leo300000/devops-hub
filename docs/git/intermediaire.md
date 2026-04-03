# 🟡 Git — Intermédiaire

## Merge vs Rebase — La grande question

!!! quote "Analogie"
    - **Merge** = Tu colles deux feuilles ensemble. L'historique montre exactement ce qui s'est passé.
    - **Rebase** = Tu réécris l'histoire comme si tu avais travaillé directement sur la branche principale. Historique plus propre.

=== "Merge"
    ```bash
    git switch main
    git merge ma-feature

    # Résultat dans l'historique :
    # * Merge branch 'ma-feature'
    # |\
    # | * Commit C (ma-feature)
    # | * Commit B (ma-feature)
    # * | Commit principal
    # |/
    # * Commit de base
    ```

=== "Rebase"
    ```bash
    git switch ma-feature
    git rebase main

    # Résultat dans l'historique :
    # * Commit C (réécrit)
    # * Commit B (réécrit)
    # * Commit principal
    # * Commit de base
    # → Historique linéaire et propre
    ```

**Règle pratique :**
- `rebase` pour ta branche feature avant de merger → historique propre
- `merge` pour intégrer dans `main` → conserve la traçabilité

---

## Cherry-pick — Prendre un seul commit

```bash
# Tu veux le commit abc123 d'une autre branche
git cherry-pick abc123

# Exemple : corriger un bug critique en prod sans merger toute la branche
git switch hotfix/prod
git cherry-pick def456  # Le commit du bugfix depuis develop
```

---

## Stash — Mettre de côté sans committer

```bash
# Tu es au milieu d'un truc, besoin de switcher de branche vite
git stash              # Sauvegarde les changements en cours
git switch autre-branche
# ... tu fais ce que t'as à faire ...
git switch ma-branche
git stash pop          # Récupère tes changements

# Gérer plusieurs stash
git stash list         # Liste les stash
git stash apply stash@{2}  # Appliquer un stash spécifique
git stash drop stash@{0}   # Supprimer un stash
```

---

## Reset — Annuler des commits

```bash
# Annuler le dernier commit (garde les changements en local)
git reset --soft HEAD~1

# Annuler le dernier commit + déstagé les fichiers
git reset HEAD~1

# Annuler le dernier commit + SUPPRIMER les changements
git reset --hard HEAD~1  # ⚠️ Irréversible !

# Annuler un fichier stagé
git restore --staged fichier.txt
```

---

## Revert — Annuler proprement (sans réécrire l'histoire)

```bash
# Crée un nouveau commit qui annule le commit abc123
# ✅ Sûr à utiliser sur les branches partagées
git revert abc123
```

| Commande | Réécrit l'histoire | Sûr sur branches partagées |
|----------|-------------------|---------------------------|
| `reset --hard` | Oui | ❌ Non |
| `revert` | Non | ✅ Oui |

---

## Résoudre les conflits

```bash
git merge feature-branch
# CONFLICT (content): Merge conflict in app.py
```

Le fichier en conflit ressemble à :
```python
<<<<<<< HEAD (main)
def greet():
    return "Bonjour"
=======
def greet():
    return "Hello"
>>>>>>> feature-branch
```

```bash
# 1. Ouvre le fichier, choisis ce que tu veux garder
# 2. Supprime les marqueurs <<<<, ====, >>>>
# 3. Stage et commit
git add app.py
git commit -m "Résout le conflit de merge"
```

---

## Tags — Marquer les releases

```bash
git tag v1.0.0                    # Tag léger
git tag -a v1.0.0 -m "Release 1.0.0"  # Tag annoté (recommandé)
git push origin v1.0.0            # Pousser le tag
git push origin --tags            # Pousser tous les tags
```

!!! success "Checkpoint intermédiaire ✅"
    Tu maîtrises : merge/rebase, cherry-pick, stash, reset/revert, conflits, tags.
