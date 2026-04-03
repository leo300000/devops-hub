# 🟢 Ansible — Débutant

!!! info "Documentation officielle"
    - [Ansible Docs](https://docs.ansible.com/ansible/latest/)
    - [Ansible Galaxy](https://galaxy.ansible.com/)
    - [Module Index](https://docs.ansible.com/ansible/latest/collections/index_module.html)

---

## C'est quoi Ansible ?

Ansible est un outil de **gestion de configuration**. Il te permet de configurer automatiquement des serveurs à distance, sans rien installer sur ces serveurs (il utilise SSH).

!!! quote "Analogie"
    Ansible, c'est comme un **chef cuisinier avec une recette**.
    La recette (le playbook) décrit exactement les étapes. Le chef (Ansible) les exécute dans l'ordre sur autant de serveurs qu'on veut, sans oubli, sans erreur humaine.

**Sans Ansible :**
```
SSH sur serveur 1 → installe nginx → configure → redémarre
SSH sur serveur 2 → installe nginx → oublie de configurer 🤦
... × 50 serveurs → erreur sur le serveur 23 → impossible de savoir laquelle
```

**Avec Ansible :**
```bash
ansible-playbook deploy-nginx.yml -i inventory.ini
# Exécuté sur 50 serveurs en parallèle, identique à chaque fois ✅
```

---

## Ansible vs les alternatives

| | Ansible | Chef | Puppet |
|--|---------|------|--------|
| **Agent sur les serveurs** | ❌ Non (SSH) | ✅ Oui | ✅ Oui |
| **Langage** | YAML | Ruby | DSL Puppet |
| **Courbe d'apprentissage** | 🟢 Facile | 🔴 Difficile | 🔴 Difficile |
| **Idempotence** | ✅ | ✅ | ✅ |

**→ Ansible = le plus simple à démarrer, rien à installer sur les serveurs cibles.**

---

## Les concepts clés

### Inventaire — C'est quoi ?

L'inventaire est le **fichier qui liste tes serveurs**. Ansible doit savoir sur quels serveurs il va travailler. Tu peux organiser tes serveurs en groupes.

!!! quote "Analogie"
    L'inventaire = ton **répertoire téléphonique**. Ansible consulte cette liste pour savoir qui appeler (via SSH).

```ini
# inventory.ini

# Groupe "webservers" — serveurs web
[webservers]
web1.example.com
web2.example.com
192.168.1.10 ansible_port=2222     # Port SSH non-standard

# Groupe "databases" — bases de données
[databases]
db1.example.com ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa

# Variables applicables à TOUS les serveurs
[all:vars]
ansible_python_interpreter=/usr/bin/python3

# Groupe de groupes
[production:children]
webservers
databases
```

---

### Playbook — C'est quoi ?

Un Playbook est le **fichier de configuration principal** d'Ansible. Il décrit **ce qu'on veut faire** sur **quels serveurs**, dans quel ordre. C'est une liste de "plays", chaque play ciblant un groupe de serveurs.

!!! quote "Analogie"
    Le Playbook = la **partition musicale**.
    Elle indique quels instruments jouent (quels serveurs), dans quel ordre, et quelle note jouer (quelle tâche exécuter).

---

### Task — C'est quoi ?

Une Task est une **action individuelle** dans un Playbook. Chaque task utilise un **module** Ansible pour faire quelque chose de précis : installer un paquet, copier un fichier, démarrer un service...

---

### Module — C'est quoi ?

Un Module est un **outil spécialisé** qu'Ansible utilise pour effectuer une tâche. Il y a un module pour installer des paquets (`apt`, `yum`), gérer des fichiers (`copy`, `file`), gérer des services (`service`), etc.

!!! quote "Analogie"
    Les modules = les **outils dans une boîte à outils**.
    `apt` = la clé à molette pour les paquets Ubuntu.
    `copy` = le pinceau pour copier des fichiers.
    `service` = l'interrupteur pour démarrer/arrêter des services.

---

## Ton premier Playbook expliqué

```yaml
# setup-webserver.yml
---                               # Début d'un fichier YAML

# Un "play" = une unité de travail sur un groupe de serveurs
- name: Configurer les serveurs web   # Nom du play (affiché pendant l'exécution)
  hosts: webservers                    # Sur quel groupe de serveurs (depuis inventory.ini)
  become: true                         # Utiliser sudo (élévation de privilèges)
  gather_facts: true                   # Collecter des infos sur les serveurs (OS, IP, RAM...)
                                       # Ces infos deviennent disponibles comme variables

  vars:                                # Variables locales à ce play
    nginx_port: 80

  tasks:                               # Liste des tâches à exécuter dans l'ordre

  # Task 1 — Mettre à jour le cache APT
  - name: Mettre à jour le cache APT
    apt:                               # Le module "apt" gère les paquets Ubuntu/Debian
      update_cache: true               # Équivalent de "apt update"
      cache_valid_time: 3600           # Ne re-télécharge pas si le cache a moins d'1h

  # Task 2 — Installer des paquets
  - name: Installer Nginx et ses dépendances
    apt:
      name:
        - nginx                        # Liste de paquets à installer
        - curl
        - git
      state: present                   # "present" = doit être installé
                                       # "absent" = doit être désinstallé
                                       # "latest" = doit être à la dernière version

  # Task 3 — Gérer un service
  - name: S'assurer que Nginx tourne
    service:                           # Le module "service" gère les services système
      name: nginx
      state: started                   # Le service doit être démarré
      enabled: true                    # Doit démarrer automatiquement au boot

  # Task 4 — Afficher une info (utile pour débugger)
  - name: Afficher l'IP du serveur configuré
    debug:
      msg: "Serveur prêt : {{ ansible_default_ipv4.address }}"
      # ansible_default_ipv4 = variable automatique collectée par gather_facts
```

```bash
# Vérifier la syntaxe sans se connecter aux serveurs
ansible-playbook setup-webserver.yml --syntax-check

# Simulation — voir ce qui va changer sans rien faire
ansible-playbook setup-webserver.yml -i inventory.ini --check

# Exécuter
ansible-playbook setup-webserver.yml -i inventory.ini

# Avec plus de détails pour débugger
ansible-playbook setup-webserver.yml -i inventory.ini -v    # Verbose
ansible-playbook setup-webserver.yml -i inventory.ini -vvv  # Très verbose
```

---

## C'est quoi l'Idempotence ?

L'idempotence signifie que tu peux lancer le même playbook **10 fois** et le résultat sera identique. Ansible **vérifie l'état actuel** avant d'agir — s'il est déjà dans l'état voulu, il ne fait rien.

```yaml
# ✅ Idempotent — Ansible vérifie si nginx est installé avant d'installer
- apt:
    name: nginx
    state: present    # "Il DOIT être présent" — pas "installe-le"

# ⚠️ PAS idempotent — s'exécute à chaque fois, même si déjà installé
- command: apt-get install -y nginx
```

La sortie d'Ansible indique :
- `ok` → Déjà dans l'état voulu, rien fait
- `changed` → Ansible a effectué un changement
- `failed` → Erreur

---

## Les modules essentiels

=== "Fichiers & dossiers"
    ```yaml
    # Copier un fichier local vers le serveur
    - name: Copier la config Nginx
      copy:
        src: fichiers/nginx.conf    # Chemin local (relatif au playbook)
        dest: /etc/nginx/nginx.conf # Chemin sur le serveur distant
        owner: root                 # Propriétaire du fichier
        group: root
        mode: '0644'               # Permissions (octal)
        backup: true               # Garde une copie avant d'écraser

    # Créer un dossier
    - name: Créer le dossier de l'app
      file:
        path: /var/www/monapp
        state: directory           # "directory" = créer comme dossier
        owner: www-data
        mode: '0755'

    # Supprimer un fichier
    - name: Supprimer le site par défaut Nginx
      file:
        path: /etc/nginx/sites-enabled/default
        state: absent              # "absent" = supprimer si existe
    ```

=== "Paquets"
    ```yaml
    # Ubuntu/Debian — Module apt
    - name: Installer des paquets
      apt:
        name:
          - nginx
          - postgresql
          - python3-pip
        state: present

    - name: Supprimer un paquet
      apt:
        name: apache2
        state: absent
        purge: true              # Supprime aussi les fichiers de config

    # CentOS/RHEL — Module yum ou dnf
    - name: Installer sur CentOS
      yum:
        name: httpd
        state: latest

    # Agnostique — Détecte automatiquement le gestionnaire de paquets
    - name: Installer peu importe l'OS
      package:
        name: git
        state: present
    ```

=== "Services"
    ```yaml
    - name: Démarrer Nginx
      service:
        name: nginx
        state: started    # started / stopped / restarted / reloaded
        enabled: true     # true = démarre au boot du serveur

    # "reloaded" = recharge la config sans couper les connexions (comme nginx -s reload)
    - name: Recharger Nginx sans downtime
      service:
        name: nginx
        state: reloaded
    ```

=== "Commandes"
    ```yaml
    # Exécuter une commande simple (sans shell)
    - name: Créer un dossier virtuel Python
      command: python3 -m venv /opt/app/venv

    # Récupérer et utiliser le résultat d'une commande
    - name: Voir la version de Python
      command: python3 --version
      register: python_version   # Stocker le résultat dans une variable

    - name: Afficher la version
      debug:
        msg: "Version : {{ python_version.stdout }}"

    # Commande shell avec pipes et redirections
    - name: Installation via pip avec log
      shell: |
        source /opt/app/venv/bin/activate
        pip install -r /opt/app/requirements.txt 2>&1 | tee /var/log/pip-install.log
    ```

---

## Ad-hoc commands — Sans playbook

Les ad-hoc commands permettent d'exécuter une seule tâche rapidement sans écrire de playbook.

```bash
# Tester la connectivité SSH vers tous les serveurs
ansible all -i inventory.ini -m ping
# web1.example.com | SUCCESS => pong
# web2.example.com | SUCCESS => pong

# Voir l'uptime de tous les serveurs web
ansible webservers -i inventory.ini -m command -a "uptime"

# Installer un paquet sur tous les serveurs (--become = sudo)
ansible all -i inventory.ini -m apt -a "name=htop state=present" --become

# Redémarrer un service
ansible webservers -i inventory.ini -m service -a "name=nginx state=restarted" --become

# Récupérer des informations système sur un serveur
ansible web1.example.com -i inventory.ini -m setup
# Affiche des centaines de variables : OS, RAM, CPU, réseau, etc.
```

---

## ansible.cfg — Configuration d'Ansible

```ini
# ansible.cfg — À placer dans le dossier du projet
[defaults]
inventory       = inventory.ini   # Inventaire par défaut
remote_user     = ubuntu          # Utilisateur SSH par défaut
private_key_file = ~/.ssh/id_rsa  # Clé SSH à utiliser
host_key_checking = False         # Ne pas demander de confirmer les nouvelles clés SSH
forks           = 10              # Nb de serveurs configurés en parallèle

[privilege_escalation]
become          = True            # Utiliser sudo par défaut
become_method   = sudo
become_user     = root
```

!!! success "Checkpoint débutant ✅"
    Tu comprends : Inventaire, Playbook, Task, Module, Idempotence.
    Tu sais : installer Ansible, écrire un playbook, utiliser les modules principaux.
