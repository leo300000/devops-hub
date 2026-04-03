# ⚙️ Ansible

> **L'électricien qui configure tes serveurs automatiquement**

---

## L'analogie parfaite

!!! quote ""
    Ansible, c'est comme une **recette de cuisine**.
    Tu écris les ingrédients et les étapes une seule fois — Ansible les exécute sur 1 ou 1000 serveurs à la fois, dans l'ordre, sans oubli.

Sans Ansible :
```
SSH sur serveur 1 → installe nginx → configure → redémarre
SSH sur serveur 2 → installe nginx → configure → redémarre
... × 50 serveurs → erreur sur le serveur 23 → rebelote 😭
```

Avec Ansible :
```bash
ansible-playbook deploy-nginx.yml -i inventory.ini
# Exécuté sur 50 serveurs en parallèle, idempotent ✅
```

---

## Ansible vs les alternatives

| Outil | Langage | Agent requis | Courbe d'apprentissage |
|-------|---------|-------------|----------------------|
| **Ansible** | YAML | ❌ Non (SSH) | 🟢 Facile |
| **Chef** | Ruby | ✅ Oui | 🔴 Difficile |
| **Puppet** | DSL Puppet | ✅ Oui | 🔴 Difficile |
| **SaltStack** | Python/YAML | ✅ Optionnel | 🟡 Moyen |

**→ Ansible = le plus simple à démarrer, sans rien installer sur les serveurs.**

---

## Niveaux disponibles

- [🟢 Débutant](debutant.md) — Inventaire, playbooks, modules de base
- [🟡 Intermédiaire](intermediaire.md) — Rôles, variables, handlers, templates
- [🔴 Avancé](avance.md) — Vault, dynamic inventory, molecule, AWX
