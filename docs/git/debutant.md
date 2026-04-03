# 🟢 Git — Débutant

## Les concepts clés

| Concept | Analogie | Explication |
|---------|----------|-------------|
| **Repository** | Dossier du projet | Contient tout le code + l'historique |
| **Commit** | Sauvegarde | Un snapshot du code à un instant T |
| **Branch** | Brouillon | Une copie parallèle pour expérimenter |
| **Remote** | Google Drive | Le repo en ligne (GitHub, GitLab...) |

---

## Configuration initiale (une seule fois)

```bash
git config --global user.name "Leo"
git config --global user.email "leo@example.com"
git config --global init.defaultBranch main
```

---

## Démarrer un projet

=== "Nouveau projet"
    ```bash
    mkdir mon-projet && cd mon-projet
    git init                    # Initialise Git dans le dossier
    ```

=== "Projet existant (clone)"
    ```bash
    git clone https://github.com/user/repo.git
    cd repo
    ```

---

## Le cycle de vie d'un fichier

```
Modifié → Staged → Commité
```

```bash
# 1. Voir l'état actuel
git status

# 2. Ajouter des fichiers au "stage" (sélectionner ce qu'on va sauvegarder)
git add fichier.txt         # Un fichier spécifique
git add src/                # Un dossier entier
git add .                   # Tout

# 3. Sauvegarder (commit)
git commit -m "Ajoute la page d'accueil"

# Ou en une ligne (add + commit pour les fichiers déjà suivis)
git commit -am "Corrige le bug du login"
```

!!! tip "Un bon message de commit"
    - ✅ `Ajoute la fonctionnalité de connexion Google`
    - ✅ `Corrige le crash sur la page profil (#42)`
    - ❌ `fix`
    - ❌ `modifications`

---

## Les branches — Travailler en parallèle

```bash
# Voir les branches
git branch

# Créer et basculer sur une nouvelle branche
git switch -c ma-nouvelle-feature
# (ancienne syntaxe : git checkout -b ma-nouvelle-feature)

# Basculer sur une branche existante
git switch main

# Supprimer une branche
git branch -d ma-feature  # Après merge
git branch -D ma-feature  # Forcer la suppression
```

!!! info "Règle d'or des branches"
    **Ne jamais travailler directement sur `main`.**
    Crée toujours une branche pour chaque feature ou bugfix.

---

## Synchroniser avec le remote (GitHub)

```bash
# Envoyer ses commits
git push origin ma-branche

# Récupérer les changements des collègues
git pull origin main

# Premier push d'une branche
git push -u origin ma-branche  # -u = définit le remote par défaut
```

---

## Voir l'historique

```bash
git log                    # Historique complet
git log --oneline          # Une ligne par commit
git log --oneline --graph  # Avec visualisation des branches
git diff                   # Changements non stagés
git diff --staged          # Changements stagés
```

---

## `.gitignore` — Ce qu'il ne faut pas suivre

```gitignore
# Fichier .gitignore
node_modules/       # Dépendances (trop lourdes)
.env                # Variables d'environnement (secrets !)
*.log               # Logs
.DS_Store           # Fichiers macOS
__pycache__/        # Cache Python
*.tfstate           # State Terraform (données sensibles)
```

!!! success "Checkpoint débutant ✅"
    Tu sais : init, add, commit, branch, push, pull. C'est 80% du Git quotidien !
