# 🟡 CI/CD — Intermédiaire

## Secrets — Variables sensibles

```yaml
# 1. Ajoute tes secrets dans : Settings → Secrets → Actions
# 2. Utilise-les dans le workflow :

steps:
- name: Deploy
  env:
    DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
    API_KEY: ${{ secrets.PROD_API_KEY }}
  run: ./deploy.sh
```

!!! warning "Jamais de secrets en dur"
    GitHub détecte et masque automatiquement les secrets dans les logs, mais ne les mets **jamais** directement dans le code YAML.

---

## Environnements — Dev / Staging / Prod

```yaml
# Créer dans : Settings → Environments → New environment
# Configurer : reviewers requis, secrets spécifiques, règles de déploiement

jobs:
  deploy-staging:
    environment: staging          # Utilise les secrets de l'env staging
    runs-on: ubuntu-latest
    steps:
    - run: echo "Deploy en staging"

  deploy-prod:
    environment: production       # Nécessite une approbation manuelle
    needs: deploy-staging         # S'exécute après staging
    runs-on: ubuntu-latest
    steps:
    - run: echo "Deploy en prod"
```

---

## Matrix — Tester sur plusieurs configs

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        python-version: ['3.10', '3.11', '3.12']
        # Génère 9 combinaisons automatiquement

    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-python@v5
      with:
        python-version: ${{ matrix.python-version }}
    - run: pytest
```

---

## Artifacts — Partager des fichiers entre jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - run: npm run build

    - name: Sauvegarder le build
      uses: actions/upload-artifact@v4
      with:
        name: dist
        path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - name: Récupérer le build
      uses: actions/download-artifact@v4
      with:
        name: dist
        path: dist/

    - run: ./deploy.sh
```

---

## Cache — Accélérer les builds

```yaml
steps:
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-

- run: pip install -r requirements.txt
```

---

## Conditions — Exécuter seulement si...

```yaml
steps:
- name: Deploy en prod
  if: github.ref == 'refs/heads/main' && github.event_name == 'push'
  run: ./deploy-prod.sh

- name: Commentaire sur PR
  if: github.event_name == 'pull_request'
  run: echo "Tests passés ✅" >> $GITHUB_STEP_SUMMARY

- name: Toujours nettoyer
  if: always()    # Même si les étapes précédentes échouent
  run: ./cleanup.sh
```

---

## Workflow CI/CD complet

```yaml
name: CI/CD Complet

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-python@v5
      with:
        python-version: '3.11'
    - run: pip install -r requirements.txt
    - run: pytest --cov=src

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Build Docker image
      run: docker build -t mon-app:${{ github.sha }} .
    - name: Push to registry
      run: |
        echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        docker push mon-app:${{ github.sha }}

  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    environment: production
    runs-on: ubuntu-latest
    steps:
    - run: echo "Déploiement en prod 🚀"
```

!!! success "Checkpoint intermédiaire ✅"
    Tu maîtrises : secrets, environnements, matrix, artifacts, cache, conditions.
