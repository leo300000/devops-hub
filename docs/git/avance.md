# 🔴 Git — Avancé

## Hooks — Automatiser les contrôles

Les hooks sont des scripts qui s'exécutent automatiquement à certains moments.

```bash
# Localisation
ls .git/hooks/
# pre-commit, commit-msg, pre-push, post-merge...
```

**Hook `pre-commit`** — Vérifier le code avant de committer :
```bash
#!/bin/sh
# .git/hooks/pre-commit
echo "🔍 Vérification du code..."
python -m pytest tests/ --quiet
if [ $? -ne 0 ]; then
  echo "❌ Tests échoués — commit annulé"
  exit 1
fi
echo "✅ Tout est bon !"
```

!!! tip "Husky — Hooks partagés avec l'équipe"
    Les hooks dans `.git/hooks` ne se commitent pas. Utilise **Husky** (JS) ou **pre-commit** (Python) pour les partager.

---

## Bisect — Trouver le commit qui a introduit un bug

```bash
# Tu sais que v1.0 fonctionnait, mais main est cassé
git bisect start
git bisect bad                  # La version actuelle est cassée
git bisect good v1.0.0          # Cette version fonctionnait

# Git te checkout un commit au milieu
# Tu testes, puis tu dis si c'est bon ou pas
git bisect good   # Ce commit est OK
git bisect bad    # Ce commit est cassé

# Git continue à bissecter jusqu'à trouver le commit fautif
# À la fin :
git bisect reset  # Revenir à l'état normal
```

---

## Reflog — Récupérer ce qu'on pensait perdu

```bash
# Le reflog garde TOUT, même les commits "supprimés"
git reflog

# abc1234 HEAD@{0}: reset: moving to HEAD~3
# def5678 HEAD@{1}: commit: Ma super feature  ← celui qu'on cherche !

# Récupérer un commit perdu
git checkout def5678
# ou
git branch recovery def5678
```

!!! info "Le reflog = le filet de sécurité ultime"
    Tant que les commits ont moins de 90 jours (config GC), ils sont récupérables.

---

## Rebase interactif — Réécrire l'histoire proprement

```bash
git rebase -i HEAD~4  # Modifier les 4 derniers commits
```

L'éditeur s'ouvre :
```
pick abc123 Ajoute la feature X
pick def456 wip  ← on va squash ça
pick ghi789 fix typo  ← et ça aussi
pick jkl012 Finalise la feature X
```

Changer en :
```
pick abc123 Ajoute la feature X
squash def456 wip
squash ghi789 fix typo
pick jkl012 Finalise la feature X
```

Résultat : 2 commits propres au lieu de 4 brouillons.

---

## Submodules — Inclure un repo dans un repo

```bash
# Ajouter un submodule
git submodule add https://github.com/user/lib.git libs/ma-lib

# Cloner un repo avec ses submodules
git clone --recurse-submodules https://github.com/user/projet.git

# Mettre à jour les submodules
git submodule update --init --recursive
```

---

## Worktrees — Plusieurs branches en même temps

```bash
# Travailler sur deux branches simultanément sans stash
git worktree add ../projet-hotfix hotfix/bug-critique

# Tu as maintenant deux dossiers :
# ./projet        → ta branche actuelle
# ../projet-hotfix → la branche hotfix

# Supprimer un worktree
git worktree remove ../projet-hotfix
```

---

## Alias utiles

```bash
git config --global alias.lg "log --oneline --graph --all --decorate"
git config --global alias.st "status -s"
git config --global alias.undo "reset --soft HEAD~1"
git config --global alias.wip "commit -am 'WIP'"

# Utilisation
git lg   # Beau graphique d'historique
git undo # Annule le dernier commit
```
