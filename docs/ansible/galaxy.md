# 🌐 Ansible Galaxy — Installer des rôles

> **Le magasin de rôles Ansible prêts à l'emploi**

---

## C'est quoi Ansible Galaxy ?

Ansible Galaxy est le **dépôt communautaire officiel** de rôles Ansible.
Plutôt que d'écrire toi-même un rôle pour installer Nginx, PostgreSQL ou Docker, tu télécharges un rôle déjà fait, testé et maintenu par la communauté.

!!! quote "Analogie"
    Ansible Galaxy = **npm** (Node) ou **pip** (Python), mais pour les rôles Ansible.
    Tu cherches un paquet, tu l'installes, tu l'utilises.

---

## Trouver un rôle

```bash
# Chercher sur la ligne de commande
ansible-galaxy search nginx
ansible-galaxy search nginx --author geerlingguy  # Filtrer par auteur

# Voir les infos d'un rôle
ansible-galaxy info geerlingguy.nginx
```

!!! tip "Jeff Geerling (geerlingguy)"
    L'auteur le plus connu de la communauté. Ses rôles sont de qualité professionnelle, testés sur toutes les distributions, et régulièrement maintenus.
    Ses rôles les plus populaires : `geerlingguy.nginx`, `geerlingguy.postgresql`, `geerlingguy.docker`, `geerlingguy.redis`

---

## 3 façons d'installer un rôle

### 1. Commande directe (simple, pour tester)

```bash
ansible-galaxy role install geerlingguy.nginx
# Le rôle est installé dans ~/.ansible/roles/

# Spécifier une version
ansible-galaxy role install geerlingguy.nginx,3.2.0

# Voir les rôles installés
ansible-galaxy list
```

### 2. Fichier `requirements.yml` (recommandé en équipe)

```yaml
# requirements.yml
---
roles:
  - name: geerlingguy.nginx
    version: "3.2.0"

  - name: geerlingguy.postgresql
    version: "3.4.1"

  - name: geerlingguy.docker
    version: "7.1.0"

  # Depuis GitHub directement
  - src: https://github.com/geerlingguy/ansible-role-redis
    name: geerlingguy.redis
    version: "1.8.0"

collections:
  - name: community.postgresql
    version: "3.4.0"
  - name: azure.azcollection
    version: "1.19.0"
```

```bash
# Installer tous les rôles listés
ansible-galaxy install -r requirements.yml

# Installer les collections aussi
ansible-galaxy collection install -r requirements.yml

# Forcer la réinstallation (si déjà installé)
ansible-galaxy install -r requirements.yml --force
```

### 3. Dans le playbook directement (collections seulement)

Pour les **rôles**, on ne peut pas les installer depuis le playbook.
Pour les **collections**, on peut les déclarer dans `galaxy.yml`.

---

## Utiliser un rôle installé

Une fois installé, on l'utilise dans le playbook avec `roles:` :

```yaml
# playbook.yml
---
- name: Configurer les serveurs web
  hosts: webservers
  become: true

  # Variables pour configurer le rôle geerlingguy.nginx
  vars:
    nginx_vhosts:
      - listen: "80"
        server_name: "monapp.example.com"
        root: "/var/www/monapp"
        index: "index.php index.html"

  # On utilise les rôles installés — pas on les installe ici
  roles:
    - geerlingguy.nginx

- name: Configurer la base de données
  hosts: databases
  become: true
  vars:
    postgresql_version: "15"
    postgresql_databases:
      - name: monapp
    postgresql_users:
      - name: monapp
        password: "{{ vault_db_password }}"
        priv: "monapp.*:ALL"
  roles:
    - geerlingguy.postgresql
```

---

## Le workflow complet en équipe

```bash
# ── Étape 1 : Un dev liste les dépendances ──
# Il crée / met à jour requirements.yml

# ── Étape 2 : Chaque dev installe les dépendances sur sa machine ──
ansible-galaxy install -r requirements.yml

# ── Étape 3 : On lance le playbook ──
ansible-playbook playbook.yml -i inventory.ini
```

!!! info "Ne pas committer `~/.ansible/roles/`"
    Les rôles s'installent dans `~/.ansible/roles/` (global) ou `./roles/` (local).
    On ne les commite pas — on commite uniquement `requirements.yml`.
    C'est le même principe que `node_modules/` : dans `.gitignore`, on installe à la demande.

---

## Résumé

| Commande | Rôle |
|----------|------|
| `ansible-galaxy search nginx` | Chercher un rôle |
| `ansible-galaxy install geerlingguy.nginx` | Installer un rôle |
| `ansible-galaxy install -r requirements.yml` | Installer depuis un fichier |
| `ansible-galaxy list` | Voir les rôles installés |
| `roles: [geerlingguy.nginx]` dans playbook | **Utiliser** le rôle |

!!! tip "Résumé en une phrase"
    `ansible-galaxy install` = **installe** le rôle sur ta machine.
    `roles:` dans le playbook = **utilise** le rôle déjà installé.
