# 🟢 CI/CD — Débutant

## Anatomie d'un workflow GitHub Actions

```yaml
# .github/workflows/mon-workflow.yml

name: Mon premier workflow           # Nom affiché sur GitHub

on:                                  # Quand se déclenche-t-il ?
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:                                # Les tâches à exécuter
  build-and-test:                    # Nom du job
    runs-on: ubuntu-latest           # Machine virtuelle

    steps:                           # Les étapes du job
    - name: Récupérer le code
      uses: actions/checkout@v4      # Action officielle

    - name: Installer les dépendances
      run: npm install               # Commande shell

    - name: Lancer les tests
      run: npm test
```

---

## Les triggers (déclencheurs)

```yaml
on:
  push:                        # À chaque push
    branches: [main, develop]
    paths:                     # Seulement si ces fichiers changent
      - 'src/**'

  pull_request:                # À chaque PR
    types: [opened, synchronize]

  schedule:                    # Planifié (cron)
    - cron: '0 6 * * 1'        # Tous les lundis à 6h

  workflow_dispatch:           # Déclenchement manuel (bouton sur GitHub)

  release:                     # Quand tu crées une release GitHub
    types: [published]
```

---

## Exemple complet — App Python

```yaml
name: CI Python

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Setup Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.11'

    - name: Installer les dépendances
      run: |
        pip install --upgrade pip
        pip install -r requirements.txt

    - name: Linter (vérification du style)
      run: flake8 src/

    - name: Tests
      run: pytest tests/ -v --tb=short

    - name: Coverage
      run: pytest --cov=src tests/ --cov-report=term-missing
```

---

## Exemple complet — App Node.js

```yaml
name: CI Node.js

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Setup Node
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'            # Cache automatique des node_modules

    - run: npm ci               # Plus strict que npm install
    - run: npm run lint
    - run: npm test
    - run: npm run build
```

---

## Voir les résultats

Sur GitHub → ton repo → onglet **Actions** → tu vois tous les workflows avec :
- ✅ Succès (vert)
- ❌ Échec (rouge)
- 🟡 En cours (jaune)

En cas d'échec, clique sur le job → sur l'étape → tu vois exactement quelle commande a planté et pourquoi.

!!! success "Checkpoint débutant ✅"
    Tu sais créer un workflow, le déclencher, et lancer des tests automatiquement.
