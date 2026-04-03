# 🟡 GitHub — Intermédiaire

## Branch Protection Rules

Settings → Branches → Add rule :

```
Branch name pattern: main

✅ Require a pull request before merging
  ✅ Require approvals: 1
  ✅ Dismiss stale PR approvals when new commits are pushed

✅ Require status checks to pass before merging
  ✅ Require branches to be up to date before merging
  Status checks: CI / test (ubuntu-latest)

✅ Require conversation resolution before merging
✅ Do not allow bypassing the above settings
```

---

## GitHub CLI — GitHub depuis le terminal

```bash
# Installation
winget install GitHub.cli  # Windows
brew install gh            # Mac

# Authentification
gh auth login

# Repos
gh repo create mon-projet --public
gh repo clone user/repo
gh repo list

# Pull Requests
gh pr create --title "Fix: bug login" --body "Closes #42"
gh pr list
gh pr view 15
gh pr merge 15 --squash

# Issues
gh issue create --title "Bug sur mobile" --label "bug"
gh issue list --label "bug"
gh issue close 42

# Actions
gh run list
gh run view 123456
gh run watch   # Voir en temps réel
```

---

## GitHub Projects — Gérer les tâches

GitHub Projects = Kanban board intégré

```
Backlog → In Progress → In Review → Done
```

Automatisations utiles :
- Quand une Issue est ouverte → ajouter dans "Backlog"
- Quand une PR est ouverte → déplacer vers "In Review"
- Quand une PR est mergée → déplacer vers "Done" et fermer l'Issue

---

## CODEOWNERS — Reviewers automatiques

```
# .github/CODEOWNERS

# Tout le code nécessite l'approbation de @leo300000
*                   @leo300000

# Les fichiers Terraform nécessitent l'équipe infra
/terraform/**       @org/infra-team

# Le CI/CD nécessite l'équipe DevOps
/.github/**         @org/devops-team

# La doc peut être validée par n'importe qui de l'équipe
/docs/**            @org/writers
```

---

## Templates — Standardiser les PRs et Issues

```markdown
<!-- .github/pull_request_template.md -->
## Description
<!-- Décris tes changements -->

## Type de changement
- [ ] 🐛 Bug fix
- [ ] ✨ Nouvelle feature
- [ ] 💥 Breaking change
- [ ] 📚 Documentation

## Checklist
- [ ] Mon code suit les conventions du projet
- [ ] J'ai ajouté des tests
- [ ] Les tests existants passent
- [ ] J'ai mis à jour la documentation

## Issues liées
Closes #
```

---

## GitHub Discussions — Forum intégré

Idéal pour :
- Questions/réponses de la communauté
- Annonces du projet
- Idées de features (avant de créer une Issue)
- Retours des utilisateurs

!!! success "Checkpoint intermédiaire ✅"
    Tu maîtrises : branch protection, GitHub CLI, Projects, CODEOWNERS, templates.
