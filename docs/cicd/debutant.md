# 🟢 CI/CD — Débutant

!!! info "Documentation officielle"
    - [GitHub Actions Docs](https://docs.github.com/en/actions)
    - [Actions Marketplace](https://github.com/marketplace?type=actions)
    - [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
    - [GitHub-hosted Runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners)

---

## Anatomie d'un workflow

```yaml
# .github/workflows/mon-workflow.yml

name: Mon premier CI                # Nom affiché sur GitHub

on:                                 # QUAND déclencher
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:                               # QUOI faire
  build-and-test:                   # Nom du job
    runs-on: ubuntu-latest          # Sur quelle machine

    steps:                          # LES ÉTAPES
    - name: Récupérer le code
      uses: actions/checkout@v4     # Action officielle GitHub

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'

    - name: Installer les dépendances
      run: npm ci

    - name: Tests
      run: npm test

    - name: Build
      run: npm run build
```

---

## Les triggers (déclencheurs)

```yaml
on:
  # Push sur des branches spécifiques
  push:
    branches:
      - main
      - develop
      - 'release/**'      # Pattern glob
    paths:
      - 'src/**'          # Seulement si src/ change
      - '!docs/**'        # Exclure docs/

  # Pull requests
  pull_request:
    branches: [main]
    types:
      - opened
      - synchronize       # Nouveau commit sur la PR
      - reopened

  # Planification (cron UTC)
  schedule:
    - cron: '0 2 * * 1'  # Tous les lundis à 2h UTC

  # Bouton manuel sur GitHub
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environnement de déploiement'
        required: true
        default: 'staging'
        type: choice
        options: [dev, staging, prod]

  # Quand une release est publiée
  release:
    types: [published]
```

---

## Exemple Python complet

```yaml
name: CI Python

on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  lint:
    name: Lint & Format
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - uses: actions/setup-python@v5
      with:
        python-version: '3.11'
        cache: 'pip'              # Cache automatique des packages pip

    - run: pip install flake8 black isort
    - run: flake8 src/ --max-line-length=100
    - run: black src/ --check
    - run: isort src/ --check-only

  test:
    name: Tests
    runs-on: ubuntu-latest
    needs: lint                   # S'exécute après lint

    services:                     # Services Docker annexes
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

    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-python@v5
      with:
        python-version: '3.11'
        cache: 'pip'

    - run: pip install -r requirements.txt

    - name: Lancer les tests avec coverage
      run: pytest tests/ -v --cov=src --cov-report=xml
      env:
        DATABASE_URL: postgresql://postgres:testpassword@localhost:5432/testdb

    - name: Upload coverage
      uses: codecov/codecov-action@v4
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
        file: ./coverage.xml
```

---

## Exemple Docker — Build et push

```yaml
name: Build Docker Image

on:
  push:
    branches: [main]
  release:
    types: [published]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Docker meta (tags automatiques)
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ghcr.io/leo300000/mon-app
        tags: |
          type=ref,event=branch
          type=semver,pattern={{version}}
          type=sha,prefix=sha-

    - name: Login GitHub Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}  # Token automatique, pas besoin de configurer

    - name: Build et push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: ${{ github.event_name != 'pull_request' }}
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

---

## Débugger un workflow qui échoue

```yaml
# Activer le mode debug
- name: Debug — voir les variables disponibles
  run: |
    echo "Branch: ${{ github.ref }}"
    echo "Event: ${{ github.event_name }}"
    echo "SHA: ${{ github.sha }}"
    env | sort

# Uploader des fichiers pour analyse
- name: Upload logs en cas d'échec
  if: failure()
  uses: actions/upload-artifact@v4
  with:
    name: test-logs
    path: |
      logs/
      *.log
    retention-days: 5
```

!!! tip "Activer les logs de debug"
    Dans Settings → Secrets → Actions, ajoute :
    - `ACTIONS_RUNNER_DEBUG` = `true`
    - `ACTIONS_STEP_DEBUG` = `true`

!!! success "Checkpoint débutant ✅"
    Tu sais créer un workflow, utiliser des triggers, tester du Python/Node, builder du Docker.
