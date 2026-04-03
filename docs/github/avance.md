# 🔴 GitHub — Avancé

## GitHub API — Automatiser tout

```bash
# Lister les repos d'un utilisateur
curl -H "Authorization: Bearer $GITHUB_TOKEN" \
  https://api.github.com/user/repos

# Créer une Issue via l'API
curl -X POST \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/leo300000/mon-repo/issues \
  -d '{"title":"Bug critique","body":"Description...","labels":["bug"]}'

# Avec gh CLI (plus simple)
gh api repos/leo300000/mon-repo/issues \
  --method POST \
  --field title="Bug critique" \
  --field body="Description..."
```

---

## GitHub Apps vs OAuth Apps vs PAT

| | PAT | OAuth App | GitHub App |
|--|-----|-----------|------------|
| **Agit en tant que** | Utilisateur | Utilisateur | App (bot) |
| **Permissions** | Larges | Larges | Fine-grained |
| **Rate limit** | 5000/h | 5000/h | 15000/h |
| **Usage** | Scripts perso | Intégrations | Automatisation prod |

```bash
# PAT fine-grained (recommandé aujourd'hui)
# Settings → Developer settings → Personal access tokens → Fine-grained
# Choisir : repo spécifique, permissions minimales, expiration
```

---

## Dependabot — Maintenir les dépendances à jour

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      dev-dependencies:
        patterns: ["@types/*", "eslint*"]

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"

  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "monthly"
```

---

## Code Scanning — Sécurité du code

```yaml
# .github/workflows/codeql.yml
name: CodeQL

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'

jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions:
      security-events: write

    steps:
    - uses: actions/checkout@v4
    - uses: github/codeql-action/init@v3
      with:
        languages: python, javascript

    - uses: github/codeql-action/autobuild@v3
    - uses: github/codeql-action/analyze@v3
```

---

## Secret Scanning & Push Protection

GitHub détecte automatiquement les secrets (API keys, tokens...) pushés par erreur.

**Activer :**
Settings → Security → Code security → Secret scanning → Enable

**Résultat :** Si tu pushe une clé AWS par erreur, GitHub :
1. Bloque le push (Push Protection)
2. T'avertit
3. Révoque automatiquement la clé chez AWS

---

## Environnements et déploiements avancés

```yaml
jobs:
  deploy:
    environment:
      name: production
      url: https://mon-app.example.com     # Lien cliquable dans GitHub

    concurrency:
      group: production                    # Un seul deploy à la fois en prod
      cancel-in-progress: false            # Ne jamais annuler un deploy prod
```
