# 🟢 GitHub — Débutant

## Le workflow de base

```
1. Fork (copier un repo)  →  Clone (télécharger en local)
2. Créer une branche      →  Modifier le code
3. Push                   →  Pull Request
4. Review                 →  Merge
```

---

## Pull Request — La base de la collaboration

Une PR = "Voici mes changements, quelqu'un peut les valider ?"

**Bonne description de PR :**
```markdown
## Ce que j'ai fait
- Ajouté la fonctionnalité de connexion Google OAuth
- Mis à jour les tests

## Comment tester
1. Cloner la branche `feature/google-oauth`
2. Lancer `npm run dev`
3. Cliquer sur "Connexion avec Google"

## Screenshots
[image avant] [image après]

## Closes #42
```

---

## Issues — Suivre les bugs et features

```markdown
## Titre : La page profil plante sur mobile

**Describe the bug**
Quand on clique sur "Modifier le profil" sur iPhone 14, l'app crash.

**To Reproduce**
1. Ouvrir l'app sur iPhone 14
2. Aller sur la page profil
3. Cliquer "Modifier"
4. → Crash

**Expected behavior**
Le formulaire d'édition doit s'ouvrir.

**Environment**
- Device: iPhone 14 Pro
- OS: iOS 17.2
- App version: 2.3.1
```

---

## Labels utiles

| Label | Couleur | Usage |
|-------|---------|-------|
| `bug` | 🔴 Rouge | Quelque chose ne fonctionne pas |
| `enhancement` | 🔵 Bleu | Nouvelle fonctionnalité |
| `documentation` | ⚪ Blanc | Documentation uniquement |
| `good first issue` | 🟢 Vert | Idéal pour les débutants |
| `priority: high` | 🟠 Orange | Urgent |

---

## Fork — Contribuer à un projet open source

```bash
# 1. Fork sur GitHub (bouton "Fork")
# 2. Clone ton fork
git clone https://github.com/TON-USERNAME/projet.git
cd projet

# 3. Ajoute le repo original comme remote
git remote add upstream https://github.com/AUTEUR-ORIGINAL/projet.git

# 4. Crée une branche, code, push
git switch -c fix/mon-bug
# ... modifications ...
git push origin fix/mon-bug

# 5. Ouvre une PR depuis ton fork vers le repo original

# 6. Garde ton fork à jour
git fetch upstream
git merge upstream/main
```

---

## GitHub Pages — Héberger un site gratuitement

```bash
# Activer GitHub Pages :
# Settings → Pages → Source → Deploy from branch → main → /docs

# Ou avec GitHub Actions (plus flexible)
# → Voir la section CI/CD
```

!!! success "Checkpoint débutant ✅"
    Tu sais créer des PRs, des Issues, forker un projet et utiliser GitHub Pages.
