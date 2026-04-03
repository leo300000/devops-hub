# 🔄 CI/CD — GitHub Actions

> **La chaîne de montage automatique de ton code**

---

## L'analogie parfaite

!!! quote ""
    CI/CD = la **chaîne de montage** d'une usine automobile.
    
    Tu poses ta pièce (ton code) sur le tapis roulant → des robots vérifient, assemblent, testent, peignent → la voiture (ton app) sort prête à livrer.

Sans CI/CD :
```
Dev → "Je déploie à la main en FTP" → Prod cassée → panique 😱
```

Avec CI/CD :
```
Dev pousse du code → Tests automatiques → Build → Déploiement → Prod stable ✅
```

---

## CI vs CD

| | CI (Intégration Continue) | CD (Déploiement Continu) |
|--|--------------------------|--------------------------|
| **Quoi** | Tester & builder automatiquement | Déployer automatiquement |
| **Quand** | À chaque push | Après la CI (ou manuellement) |
| **But** | Détecter les bugs tôt | Livrer vite, souvent |

---

## GitHub Actions vs les alternatives

| Outil | Hébergement | Intégration GitHub | Gratuit |
|-------|-------------|-------------------|---------|
| **GitHub Actions** | Cloud | ⭐⭐⭐⭐⭐ Native | 2000 min/mois |
| **GitLab CI** | Cloud/Self | ⭐⭐⭐ Bonne | 400 min/mois |
| **Jenkins** | Self-hosted | ⭐⭐ Plugin | Oui (infra à payer) |
| **CircleCI** | Cloud | ⭐⭐⭐⭐ Bonne | 6000 min/mois |

**→ Si tu es sur GitHub, GitHub Actions est le choix naturel.**

---

## Niveaux disponibles

- [🟢 Débutant](debutant.md) — Premier workflow, triggers, jobs
- [🟡 Intermédiaire](intermediaire.md) — Secrets, environnements, matrix, artifacts
- [🔴 Avancé](avance.md) — Reusable workflows, custom actions, OIDC, sécurité
