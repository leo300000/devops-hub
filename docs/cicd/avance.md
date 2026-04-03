# 🔴 CI/CD — Avancé

## Reusable Workflows — DRY pour les pipelines

```yaml
# .github/workflows/reusable-test.yml
on:
  workflow_call:                   # Ce workflow peut être appelé
    inputs:
      python-version:
        required: true
        type: string
    secrets:
      CODECOV_TOKEN:
        required: true

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-python@v5
      with:
        python-version: ${{ inputs.python-version }}
    - run: pytest --cov
```

```yaml
# .github/workflows/ci.yml — Appeler le workflow réutilisable
jobs:
  tests:
    uses: ./.github/workflows/reusable-test.yml
    with:
      python-version: '3.11'
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

---

## Custom Actions — Créer ses propres actions

=== "Composite Action"
    ```yaml
    # .github/actions/setup-project/action.yml
    name: 'Setup Project'
    description: 'Installe et configure le projet'
    inputs:
      python-version:
        description: 'Version Python'
        default: '3.11'

    runs:
      using: composite
      steps:
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python-version }}
      - run: pip install -r requirements.txt
        shell: bash
    ```
    
    ```yaml
    # Utilisation dans un workflow
    steps:
    - uses: ./.github/actions/setup-project
      with:
        python-version: '3.12'
    ```

=== "Docker Action"
    ```dockerfile
    # .github/actions/mon-outil/Dockerfile
    FROM python:3.11-alpine
    COPY entrypoint.sh /entrypoint.sh
    ENTRYPOINT ["/entrypoint.sh"]
    ```

---

## OIDC — Authentification sans secrets

Plutôt que de stocker des credentials Azure dans les secrets GitHub :

```yaml
jobs:
  deploy:
    permissions:
      id-token: write      # Nécessaire pour OIDC
      contents: read

    steps:
    - name: Login Azure via OIDC (sans secret !)
      uses: azure/login@v2
      with:
        client-id: ${{ secrets.AZURE_CLIENT_ID }}
        tenant-id: ${{ secrets.AZURE_TENANT_ID }}
        subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
        # Pas besoin de client_secret ! GitHub génère un token temporaire
```

!!! tip "OIDC = meilleure sécurité"
    Le token est temporaire (15 min), spécifique à ce workflow, et auditable. Bien supérieur à un secret statique.

---

## Sécurité des workflows

```yaml
# ✅ Toujours épingler les actions par SHA (pas par tag)
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4.2.2

# ✅ Limiter les permissions
permissions:
  contents: read          # Accès minimum nécessaire

# ✅ Ne pas utiliser pull_request_target avec du code non vérifié
# ❌ Dangereux :
on:
  pull_request_target:
    # Donne accès aux secrets même sur les PRs externes !
```

---

## Optimisations avancées

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    
    # Concurrency — Annule les runs précédents sur la même branche
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
      cancel-in-progress: true

    steps:
    # Checkout minimal (sans tout l'historique)
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0    # 0 = tout, 1 = dernier commit seulement

    # Jobs en parallèle
    - run: npm run lint &
           npm run typecheck &
           wait
```

---

## Résumé dans les PR

```yaml
steps:
- name: Résultats des tests
  if: always()
  run: |
    echo "## Résultats CI 🚀" >> $GITHUB_STEP_SUMMARY
    echo "| Check | Status |" >> $GITHUB_STEP_SUMMARY
    echo "|-------|--------|" >> $GITHUB_STEP_SUMMARY
    echo "| Tests | ✅ Passé |" >> $GITHUB_STEP_SUMMARY
    echo "| Coverage | 87% |" >> $GITHUB_STEP_SUMMARY
```
