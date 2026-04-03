# 🟢 Git — Débutant

!!! info "Documentation officielle"
    - [Git Book (fr)](https://git-scm.com/book/fr/v2) — La référence complète
    - [Git Reference](https://git-scm.com/docs)
    - [Learn Git Branching](https://learngitbranching.js.org/?locale=fr_FR) — Exercices interactifs visuels
    - [Oh Shit, Git!](https://ohshitgit.com/fr) — Sortir des situations difficiles

---

## Installation et configuration

```bash
# Windows
winget install Git.Git

# Mac
brew install git

# Ubuntu
sudo apt install git

# Configuration initiale (obligatoire)
git config --global user.name "Leo"
git config --global user.email "leo@example.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"  # VS Code comme éditeur

# Voir sa config
git config --list
```

---

## Les concepts clés

| Concept | Analogie | Explication |
|---------|----------|-------------|
| **Repository** | Dossier du projet | Contient tout le code + l'historique |
| **Working Directory** | Ta table de travail | Fichiers que tu édites |
| **Staging Area** | La corbeille de courrier | Fichiers prêts à être commités |
| **Commit** | Sauvegarde | Snapshot du code à un instant T |
| **Branch** | Brouillon parallèle | Copie pour expérimenter sans risque |
| **Remote** | Serveur (GitHub) | Où le code est partagé/sauvegardé |

### Le cycle de vie d'un fichier

```
Untracked → Staged → Committed → Pushed
   (nouveau)   (git add)  (git commit)  (git push)
```

---

## Démarrer un projet

=== "Nouveau projet"
    ```bash
    mkdir mon-projet && cd mon-projet
    git init                       # Crée un repo Git local
    # Crée un dossier .git/ invisible qui contient tout l'historique
    ```

=== "Cloner un projet existant"
    ```bash
    git clone https://github.com/user/repo.git
    cd repo
    # ou dans un dossier spécifique
    git clone https://github.com/user/repo.git mon-dossier
    ```

---

## Les commandes du quotidien

```bash
# === VOIR l'état ===
git status                   # Fichiers modifiés, stagés, non-suivis
git status -s                # Version courte
# ?? fichier.txt             → non-suivi
# M  fichier.txt             → modifié et stagé
#  M fichier.txt             → modifié non-stagé

git diff                     # Changements non-stagés
git diff --staged            # Changements stagés (ce qui va être commité)
git log                      # Historique complet
git log --oneline            # Une ligne par commit
git log --oneline --graph --all  # Visualisation des branches
git log -p                   # Historique avec les diffs

# === STAGE des fichiers ===
git add fichier.txt          # Un fichier
git add src/                 # Un dossier entier
git add *.js                 # Par pattern
git add .                    # Tout (attention aux fichiers sensibles !)
git add -p                   # Interactif — choisir les parties à stager

# === COMMITTER ===
git commit -m "Ajoute la page de connexion"
git commit                   # Ouvre l'éditeur pour écrire un message long
git commit -am "Fix bug"     # add + commit pour les fichiers déjà suivis
git commit --amend           # Modifier le dernier commit (message ou contenu)
```

---

## Écrire de bons messages de commit

Suit la convention [Conventional Commits](https://www.conventionalcommits.org/fr/) :

```
type(scope): description courte

Corps optionnel — explication du POURQUOI

Closes #42
```

**Types :**

| Type | Usage |
|------|-------|
| `feat` | Nouvelle fonctionnalité |
| `fix` | Correction de bug |
| `docs` | Documentation uniquement |
| `style` | Formatage, pas de changement de logique |
| `refactor` | Refactoring sans new feature ni bug fix |
| `test` | Ajout ou modification de tests |
| `chore` | Maintenance, dépendances |
| `ci` | Configuration CI/CD |

```bash
# ✅ Bons exemples
git commit -m "feat(auth): ajoute la connexion via Google OAuth"
git commit -m "fix(api): corrige le crash sur la route /users quand id invalide"
git commit -m "docs: met à jour le README avec les instructions d'installation"

# ❌ Mauvais exemples
git commit -m "fix"
git commit -m "wip"
git commit -m "modifs du 3 avril"
```

---

## Branches — Travailler en parallèle

```bash
# Voir les branches
git branch                   # Locales
git branch -a                # Locales + distantes

# Créer et basculer (méthode moderne)
git switch -c feature/connexion-google
# (ancienne méthode)
git checkout -b feature/connexion-google

# Basculer sur une branche existante
git switch main
git switch develop

# Renommer une branche
git branch -m ancien-nom nouveau-nom

# Supprimer une branche
git branch -d feature/connexion-google   # Seulement si mergée
git branch -D feature/connexion-google   # Forcer la suppression
```

### Convention de nommage

```
main              → code en production
develop           → intégration des features
feature/xxx       → nouvelle fonctionnalité
fix/xxx           → correction de bug
hotfix/xxx        → correction urgente en prod
release/x.y.z     → préparation d'une release
```

---

## Synchroniser avec GitHub

```bash
# Envoyer ses commits sur GitHub
git push origin ma-branche

# Premier push d'une branche (crée le remote tracking)
git push -u origin ma-branche
# Ensuite, tu peux juste faire git push

# Récupérer les derniers changements
git pull origin main         # fetch + merge
git pull --rebase origin main  # fetch + rebase (historique plus propre)

# Récupérer sans merger
git fetch origin             # Télécharge mais ne merge pas
git fetch --all              # Toutes les branches distantes
```

---

## `.gitignore` — Ce qu'il ne faut jamais versionner

```gitignore
# Dépendances
node_modules/
vendor/
__pycache__/
*.pyc
.venv/

# Build
dist/
build/
*.egg-info/
site/

# Environnement & secrets — JAMAIS commiter ça !
.env
.env.local
.env.production
*.key
*.pem
secrets/

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store       # macOS
Thumbs.db       # Windows

# Terraform
.terraform/
*.tfstate
*.tfstate.backup
*.tfvars        # Si contient des secrets

# Logs
*.log
logs/
```

!!! tip "gitignore.io"
    Va sur [gitignore.io](https://www.toptal.com/developers/gitignore) pour générer automatiquement un `.gitignore` adapté à ta stack.

---

## Voir qui a écrit quoi — `git blame`

```bash
# Voir qui a écrit chaque ligne d'un fichier
git blame fichier.py

# 3b5f8c2 (Leo 2024-01-15 14:32:11) def calculate():
# a1d9f4e (Alice 2024-01-20 09:15:33)     result = x + y
```

!!! success "Checkpoint débutant ✅"
    Tu sais : installer Git, init/clone, add/commit, branches, push/pull, .gitignore, git log.
