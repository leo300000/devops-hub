# 🟢 CI/CD — Débutant

!!! info "Documentation officielle"
    - [GitHub Actions Docs](https://docs.github.com/en/actions)
    - [Actions Marketplace](https://github.com/marketplace?type=actions)
    - [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

---

## C'est quoi le CI/CD ?

**CI (Continuous Integration)** = Intégration Continue.
À chaque fois qu'un développeur pousse du code, une série de vérifications automatiques se déclenche : tests, analyse du code, build...

**CD (Continuous Delivery/Deployment)** = Déploiement Continu.
Après que la CI valide le code, il est automatiquement déployé en staging, puis en production.

!!! quote "Analogie"
    CI/CD = la **chaîne de montage** d'une usine automobile.
    Tu poses ta pièce (ton code) sur le tapis roulant. Des robots vérifient automatiquement chaque étape (tests, qualité, sécurité). Si tout est bon, la voiture (ton app) sort prête à être livrée — sans intervention humaine.

**Sans CI/CD :**
```
Dev code → "Je déploie à la main" → Oublie de lancer les tests
→ Bug en prod → Panique le vendredi soir à 17h 😱
```

**Avec CI/CD :**
```
Dev pousse du code → Tests automatiques → Build → Deploy
→ Si tests échouent : bloqué, jamais en prod ✅
→ Si tests passent : déployé proprement, automatiquement ✅
```

---

## C'est quoi GitHub Actions ?

GitHub Actions est le système CI/CD **intégré dans GitHub**. Tu crées des fichiers YAML dans `.github/workflows/` et GitHub les exécute automatiquement selon les déclencheurs que tu configures.

!!! quote "Analogie"
    GitHub Actions = un **robot employé** qui surveille ton repo. Tu lui donnes des instructions ("quand quelqu'un pousse du code, lance ces commandes"), et il les exécute fidèlement.

---

## Les concepts clés

### Workflow — C'est quoi ?

Un Workflow est le **fichier de configuration** global. Il définit quand et quoi faire. Un repo peut avoir plusieurs workflows.

### Job — C'est quoi ?

Un Job est un **groupe d'étapes** qui s'exécutent sur une même machine. Plusieurs jobs peuvent tourner en parallèle (ou en séquence avec `needs:`).

### Step — C'est quoi ?

Un Step est une **étape individuelle** dans un job : une commande shell ou une Action. Les steps s'exécutent dans l'ordre, sur la même machine.

### Action — C'est quoi ?

Une Action est un **bloc réutilisable** créé par la communauté (ou toi). Au lieu de réécrire `git clone` et l'installation de Node à chaque fois, tu utilises `actions/checkout@v4` et `actions/setup-node@v4`.

### Runner — C'est quoi ?

Un Runner est la **machine virtuelle** sur laquelle le job s'exécute. GitHub fournit des runners gratuits (`ubuntu-latest`, `windows-latest`, `macos-latest`).

---

## Anatomie d'un workflow expliquée ligne par ligne

```yaml
# Chemin obligatoire : .github/workflows/mon-workflow.yml

name: CI Mon Application        # Nom affiché dans l'onglet Actions de GitHub

# ─── QUAND ce workflow se déclenche ───
on:
  push:                         # Quand quelqu'un fait git push
    branches: [main]            # Uniquement sur la branche main
  pull_request:                 # Quand une PR est ouverte/mise à jour
    branches: [main]            # Ciblant la branche main

# ─── QUOI faire ───
jobs:

  # "test" est le nom du job (tu choisis)
  test:
    runs-on: ubuntu-latest      # La machine virtuelle à utiliser (Ubuntu gratuit)

    # ─── Les étapes du job (dans l'ordre) ───
    steps:

    # Step 1 — Récupérer le code du repo sur la machine virtuelle
    - name: Récupérer le code
      uses: actions/checkout@v4
      # "uses" = utiliser une Action existante de la marketplace
      # actions/checkout = l'Action officielle GitHub pour cloner le repo

    # Step 2 — Installer Node.js sur la machine virtuelle
    - name: Installer Node.js
      uses: actions/setup-node@v4
      with:                     # Paramètres de l'Action
        node-version: '20'
        cache: 'npm'            # Cache les node_modules entre les runs (plus rapide)

    # Step 3 — Commande shell classique
    - name: Installer les dépendances
      run: npm ci               # "run" = exécuter une commande shell
      # npm ci = comme npm install mais plus strict (respecte le package-lock.json)

    # Step 4
    - name: Lancer les tests
      run: npm test

    # Step 5 — Plusieurs commandes avec |
    - name: Build et vérifications
      run: |
        npm run lint
        npm run build
        echo "Build terminé !"
```

---

## Les triggers (déclencheurs) expliqués

```yaml
on:
  # Déclenche sur un push
  push:
    branches: [main, develop]
    paths:                      # Seulement si ces fichiers changent
      - 'src/**'                # Tout dans src/
      - '!docs/**'              # Sauf docs/ (le ! = exclusion)

  # Déclenche sur une PR
  pull_request:
    branches: [main]
    types:
      - opened                  # Quand la PR est créée
      - synchronize             # Quand un nouveau commit est poussé sur la PR

  # Planifié — Cron (UTC)
  schedule:
    - cron: '0 2 * * 1'         # Tous les lundis à 2h UTC
    # Format : minute heure jour-du-mois mois jour-de-semaine

  # Bouton manuel dans l'interface GitHub
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environnement cible'
        required: true
        default: 'staging'
        type: choice
        options: [dev, staging, prod]

  # Quand une release GitHub est publiée
  release:
    types: [published]
```

---

## Exemple complet — App Python avec tests et base de données

```yaml
name: CI Python

on:
  push:
    branches: [main]
  pull_request:

jobs:
  # Job 1 : Vérifier la qualité du code
  lint:
    name: Qualité du code
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - uses: actions/setup-python@v5
      with:
        python-version: '3.11'
        cache: 'pip'

    - name: Installer les outils de lint
      run: pip install flake8 black

    - name: Vérifier le style (flake8)
      run: flake8 src/ --max-line-length=100
      # Échoue si le code ne respecte pas les conventions

    - name: Vérifier le formatage (black)
      run: black src/ --check
      # Échoue si le code n'est pas formaté avec black

  # Job 2 : Lancer les tests (s'exécute APRÈS lint)
  test:
    name: Tests
    runs-on: ubuntu-latest
    needs: lint                 # Attend que le job "lint" réussisse

    # Services Docker qui tournent à côté du job
    # Utile pour lancer une vraie base de données pendant les tests
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: testpassword
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
          # K8s attend que Postgres soit prêt avant de lancer les steps

    steps:
    - uses: actions/checkout@v4

    - uses: actions/setup-python@v5
      with:
        python-version: '3.11'
        cache: 'pip'

    - run: pip install -r requirements.txt

    - name: Lancer les tests avec couverture
      run: pytest tests/ -v --cov=src --cov-report=xml
      env:
        DATABASE_URL: postgresql://postgres:testpassword@localhost:5432/testdb
        # La variable d'environnement est disponible dans cette step
```

---

## Exemple — Build et push d'une image Docker

```yaml
name: Docker Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    # Se connecter au GitHub Container Registry (GHCR)
    # GITHUB_TOKEN est un token automatique, pas besoin de le configurer
    - name: Login GHCR
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}    # Ton username GitHub
        password: ${{ secrets.GITHUB_TOKEN }}

    # Builder et pusher l'image
    - name: Build et push
      uses: docker/build-push-action@v5
      with:
        context: .                        # Le dossier contenant le Dockerfile
        push: true                        # Pousser l'image (false = juste builder)
        tags: ghcr.io/leo300000/mon-app:latest
        cache-from: type=gha             # Cache GitHub Actions (builds plus rapides)
        cache-to: type=gha,mode=max
```

---

## Voir les résultats sur GitHub

1. Va sur ton repo GitHub
2. Clique sur l'onglet **Actions**
3. Tu vois tous les workflows avec leur statut :
   - ✅ Vert = succès
   - ❌ Rouge = échec
   - 🟡 Jaune = en cours

En cas d'échec : clique sur le job → sur la step qui a échoué → tu vois exactement quelle commande a planté et pourquoi.

!!! tip "Débugger un workflow"
    Ajoute une step de debug pour voir les variables disponibles :
    ```yaml
    - name: Debug
      run: |
        echo "Branch: ${{ github.ref }}"
        echo "Event: ${{ github.event_name }}"
        env | sort
    ```

!!! success "Checkpoint débutant ✅"
    Tu comprends : Workflow, Job, Step, Action, Runner, Trigger.
    Tu sais créer un CI pour Python et Docker.
