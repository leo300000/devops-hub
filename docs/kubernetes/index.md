# 🐳 Kubernetes

> **Le chef d'orchestre de tes containers**

---

## L'analogie parfaite

!!! quote ""
    Docker = un musicien qui joue seul.
    Kubernetes = le chef d'orchestre qui coordonne 100 musiciens.

Sans Kubernetes :
```
Container 1 plante → l'app est down → quelqu'un le redémarre manuellement à 3h du matin 😴
```

Avec Kubernetes :
```
Container 1 plante → K8s le redémarre automatiquement en 2 secondes → personne ne s'en rend compte ✅
```

---

## Docker vs Kubernetes

| | Docker (seul) | Kubernetes |
|--|--------------|------------|
| **Scale** | Manuel (`docker run` x fois) | Automatique |
| **Restart** | Manuel | Automatique |
| **Load balancing** | Non inclus | Intégré |
| **Updates** | Downtime | Rolling updates sans downtime |
| **Nb de containers** | Dizaines | Des milliers |

---

## Les objets K8s à retenir

| Objet | Analogie | Rôle |
|-------|----------|------|
| **Pod** | Un container (ou groupe) | Plus petite unité K8s |
| **Deployment** | Recette de pods | Gère le nb de replicas |
| **Service** | Point d'entrée réseau | Expose l'app |
| **Ingress** | Réceptionniste | Route le trafic HTTP/S |
| **ConfigMap** | Fichier de config | Variables non-sensibles |
| **Secret** | Coffre-fort | Mots de passe, tokens |

---

## Niveaux disponibles

- [🟢 Débutant](debutant.md) — Pods, Deployments, Services
- [🟡 Intermédiaire](intermediaire.md) — Ingress, ConfigMaps, Secrets, HPA
- [🔴 Avancé](avance.md) — RBAC, Operators, Helm avancé, réseau
