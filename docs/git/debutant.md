# 🟢 Git — Débutant

!!! info "Documentation officielle"
    - [Git Book (fr)](https://git-scm.com/book/fr/v2)
    - [Learn Git Branching](https://learngitbranching.js.org/?locale=fr_FR) — Exercices visuels interactifs
    - [Oh Shit, Git!](https://ohshitgit.com/fr) — Sortir des situations difficiles
    - [Conventional Commits](https://www.conventionalcommits.org/fr/)

---

## C'est quoi Git ?

Git est un **système de contrôle de version** (VCS). Il enregistre l'historique de chaque modification de ton code, qui l'a faite, quand, et pourquoi.

!!! quote "Analogie"
    Git, c'est la **machine à remonter le temps** de ton code.
    À chaque "sauvegarde" (commit), Git prend un snapshot complet. Tu peux revenir à n'importe quel moment dans le passé, comparer deux versions, ou travailler sur plusieurs idées en parallèle sans les mélanger.

**Sans Git :**
```
projet_final.zip
projet_final_v2.zip
projet_final_v2_VRAI.zip
projet_final_v2_VRAI_cette_fois.zip
projet_final_v2_VRAI_cette_fois_2.zip   😱
```

**Avec Git :**
```bash
git log --oneline
# a1b2c3d Ajoute la fonctionnalité de connexion
# e4f5g6h Corrige le bug du panier
# i7j8k9l Version initiale
```

---

## Les 4 zones de Git — Comment ça marche

Comprendre ces 4 zones est essentiel. Beaucoup de commandes Git déplacent des fichiers d'une zone à l'autre.

```
┌─────────────────┐   git add    ┌──────────────┐   git commit  ┌──────────────┐   git push   ┌──────────────┐
│ Working         │ ──────────→  │ Staging Area │ ───────────→  │ Local Repo   │ ──────────→  │ Remote Repo  │
│ Directory       │              │ (Index)      │               │ (.git/)      │              │ (GitHub)     │
│                 │ ←──────────  │              │ ←───────────  │              │ ←──────────  │              │
│ Tes fichiers    │  git restore │              │  git reset    │ Tous les     │  git pull    │ Code partagé │
│ en cours        │              │ Fichiers     │               │ commits      │              │ avec l'équipe│
│ d'édition       │              │ prêts à être │               │              │              │              │
└─────────────────┘              │ commités     │               └──────────────┘              └──────────────┘
                                 └──────────────┘
```

### Working Directory — C'est quoi ?

C'est tout simplement **ton dossier de travail** — les fichiers que tu vois et édites. Git surveille les changements mais n'enregistre rien automatiquement.

### Staging Area (Index) — C'est quoi ?

La Staging Area est une **zone de préparation** entre tes modifications et le commit. Tu choisis exactement quels changements inclure dans le prochain commit — tu n'es pas obligé de tout commiter d'un coup.

!!! quote "Analogie"
    La Staging Area = ton **caddie de supermarché**.
    Tu fais tes courses (modifies des fichiers), tu mets certains articles dans le caddie (`git add`), et tu passes en caisse (`git commit`) seulement ce qui est dans le caddie.

### Local Repository — C'est quoi ?

Le dossier `.git/` à la racine de ton projet. Il contient **tout l'historique** — chaque commit, chaque branche, chaque tag. C'est la "base de données" de Git sur ta machine.

### Remote Repository — C'est quoi ?

Le repo **en ligne** (sur GitHub, GitLab...). Il permet de partager le code avec l'équipe et de sauvegarder hors de ta machine.

---

## Installation et configuration

```bash
# Installation
winget install Git.Git   # Windows
brew install git         # Mac
sudo apt install git     # Ubuntu

# Configuration obligatoire (une seule fois)
git config --global user.name "Leo"
git config --global user.email "leo@example.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"   # VS Code comme éditeur Git
```

---

## Démarrer un projet

=== "Nouveau projet"
    ```bash
    mkdir mon-projet && cd mon-projet
    git init
    # Crée le dossier .git/ invisible
    # → ton dossier est maintenant un repo Git
    ```

=== "Cloner un projet existant"
    ```bash
    # Récupère le repo + tout l'historique
    git clone https://github.com/user/repo.git
    cd repo
    ```

---

## Les commandes du quotidien

### `git status` — Voir l'état actuel

```bash
git status

# ?? fichier.txt      → Nouveau fichier, non-suivi par Git
# M  fichier.txt      → Modifié et dans la Staging Area (prêt à committer)
#  M fichier.txt      → Modifié mais pas encore dans la Staging Area
# D  fichier.txt      → Supprimé

git status -s   # Version courte
```

### `git add` — Ajouter à la Staging Area

```bash
git add fichier.txt          # Un fichier spécifique
git add src/                 # Tout un dossier
git add *.js                 # Tous les fichiers .js
git add .                    # Tout (attention aux fichiers sensibles !)
git add -p                   # Mode interactif — choisir partie par partie ce qu'on stage
```

### `git commit` — Sauvegarder un snapshot

```bash
git commit -m "feat: ajoute la page de connexion"
git commit                   # Ouvre l'éditeur pour un message long
git commit -am "fix: corrige le crash"  # add + commit (fichiers déjà suivis seulement)
```

### `git diff` — Voir les changements

```bash
git diff                     # Changements dans le Working Directory (non-stagés)
git diff --staged            # Changements dans la Staging Area (qui vont être commités)
git diff main..ma-branche    # Différence entre deux branches
```

---

## C'est quoi un Commit ?

Un Commit est un **snapshot (photo) de l'état de ton code** à un instant T. Chaque commit a :
- Un **hash** unique (ex: `a1b2c3d`) qui l'identifie
- Un **message** qui explique pourquoi ce changement a été fait
- Un **auteur** et une **date**
- Un **parent** (le commit précédent)

!!! tip "Écrire de bons messages de commit — Conventional Commits"
    La convention [Conventional Commits](https://www.conventionalcommits.org/fr/) structure les messages :
    
    ```
    type(scope): description courte
    
    Corps optionnel (le POURQUOI, pas le QUOI)
    
    Closes #42
    ```
    
    | Type | Usage |
    |------|-------|
    | `feat` | Nouvelle fonctionnalité |
    | `fix` | Correction de bug |
    | `docs` | Documentation |
    | `refactor` | Refactoring sans changement fonctionnel |
    | `test` | Ajout/modification de tests |
    | `ci` | CI/CD |
    | `chore` | Maintenance |

    ```bash
    # ✅ Bons
    git commit -m "feat(auth): ajoute la connexion Google OAuth"
    git commit -m "fix(api): corrige le crash quand userId est null"
    git commit -m "docs: ajoute les instructions d'installation Docker"
    
    # ❌ Mauvais
    git commit -m "fix"
    git commit -m "wip"
    git commit -m "modifications du 3 avril"
    ```

---

## C'est quoi une Branche ?

Une branche est une **ligne de développement parallèle** et indépendante. Tu peux créer une branche pour développer une feature sans impacter le code principal. Si ça ne marche pas, tu supprimes la branche — le code principal est intact.

!!! quote "Analogie"
    La branche principale (`main`) = la **route nationale**.
    Créer une branche = prendre une **bretelle** pour faire un détour. Tu reviens sur la route nationale quand tu as fini (merge).

```
main :     A → B → C → F (merge)
                    ↘
feature :           D → E
```

```bash
# Voir les branches
git branch                          # Branches locales (* = branche actuelle)
git branch -a                       # Locales + distantes

# Créer et basculer sur une nouvelle branche
git switch -c feature/connexion-google
# Ancienne syntaxe (toujours valide) :
git checkout -b feature/connexion-google

# Basculer sur une branche existante
git switch main

# Supprimer une branche
git branch -d feature/connexion-google   # Seulement si mergée
git branch -D feature/connexion-google   # Forcer (attention !)
```

### Convention de nommage des branches

```
main              → code en production
develop           → intégration des features
feature/xxx       → nouvelle fonctionnalité
fix/xxx           → correction de bug
hotfix/xxx        → correction urgente en prod
release/1.2.0     → préparation d'une release
```

---

## Synchroniser avec GitHub

### `git remote` — C'est quoi ?

Un remote est l'**adresse du repo en ligne**. Par convention, le remote principal s'appelle `origin`.

```bash
# Voir les remotes configurés
git remote -v
# origin  https://github.com/user/repo.git (fetch)
# origin  https://github.com/user/repo.git (push)

# Ajouter un remote
git remote add origin https://github.com/user/repo.git
```

```bash
# Envoyer ses commits sur GitHub
git push origin ma-branche

# Premier push d'une branche (configure le tracking)
git push -u origin ma-branche
# Ensuite : git push suffit

# Récupérer les derniers changements (fetch + merge)
git pull origin main

# Récupérer les changements sans merger
git fetch origin
git fetch --all   # Toutes les branches distantes
```

---

## `.gitignore` — Ce qu'il ne faut JAMAIS versionner

```gitignore
# Dépendances (trop lourdes, régénérables)
node_modules/
vendor/
__pycache__/
.venv/
*.pyc

# Build (régénérable)
dist/
build/
site/

# Secrets — JAMAIS dans Git !
.env
.env.local
.env.production
*.key
*.pem
secrets/
credentials.json

# IDE (propre à chaque développeur)
.vscode/
.idea/
*.swp

# OS
.DS_Store        # macOS
Thumbs.db        # Windows

# Terraform
.terraform/
*.tfstate
*.tfstate.backup

# Logs
*.log
logs/
```

!!! tip "gitignore.io"
    [gitignore.io](https://www.toptal.com/developers/gitignore) génère automatiquement un `.gitignore` selon ta stack (Python, Node, Terraform, etc.)

---

## Voir l'historique

```bash
git log                        # Historique complet
git log --oneline              # Une ligne par commit
git log --oneline --graph --all  # Avec graphe des branches
git log -p                     # Avec les diffs de chaque commit
git log --author="Leo"         # Commits d'un auteur spécifique
git log --since="2024-01-01"   # Depuis une date
git log fichier.txt            # Historique d'un fichier
git show a1b2c3d               # Détails d'un commit spécifique
git blame fichier.txt          # Qui a écrit chaque ligne
```

!!! success "Checkpoint débutant ✅"
    Tu comprends : les 4 zones (Working Dir, Staging, Local Repo, Remote), commits, branches, remotes.
    Tu sais : init/clone, add/commit, push/pull, .gitignore, git log.
