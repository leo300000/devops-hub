# 🟢 Ansible — Débutant

!!! info "Documentation officielle"
    - [Ansible Docs](https://docs.ansible.com/ansible/latest/)
    - [Ansible Galaxy](https://galaxy.ansible.com/) — Roles communautaires
    - [Collection AzureRM](https://galaxy.ansible.com/ui/repo/published/azure/azcollection/)
    - [Module Index](https://docs.ansible.com/ansible/latest/collections/index_module.html)

---

## Installation

```bash
# Ubuntu/Debian
sudo apt update && sudo apt install ansible -y

# Mac
brew install ansible

# Via pip (recommandé pour avoir la dernière version)
pip install ansible

# Vérifier
ansible --version
# ansible [core 2.16.0]
```

---

## L'inventaire — Tes serveurs

```ini
# inventory.ini

# Groupe de serveurs web
[webservers]
web1.example.com
web2.example.com
192.168.1.10 ansible_port=2222  # Port SSH custom

# Groupe de bases de données
[databases]
db1.example.com ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa

# Variables pour tous les serveurs
[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_common_args='-o StrictHostKeyChecking=no'

# Groupe de groupes
[production:children]
webservers
databases
```

```yaml
# inventory.yml (format YAML — plus lisible pour des grandes infras)
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
          ansible_port: 2222
    databases:
      hosts:
        db1.example.com:
          ansible_user: ubuntu
  vars:
    ansible_python_interpreter: /usr/bin/python3
```

---

## Ton premier playbook

```yaml
# setup-webserver.yml
---
- name: Configurer les serveurs web
  hosts: webservers          # Groupe de l'inventaire
  become: true               # Sudo (équivalent de sudo su)
  gather_facts: true         # Collecte infos sur les serveurs (OS, IP, etc.)

  vars:
    nginx_port: 80
    app_user: www-data

  tasks:

  - name: Mettre à jour le cache APT
    apt:
      update_cache: true
      cache_valid_time: 3600  # Valide 1h, évite de re-télécharger à chaque run

  - name: Installer les paquets nécessaires
    apt:
      name:
        - nginx
        - curl
        - git
        - python3-pip
      state: present

  - name: S'assurer que Nginx est démarré et activé au boot
    service:
      name: nginx
      state: started
      enabled: true

  - name: Afficher l'IP du serveur
    debug:
      msg: "Serveur configuré : {{ ansible_default_ipv4.address }}"
```

```bash
# Vérifier la syntaxe sans se connecter
ansible-playbook setup-webserver.yml --syntax-check

# Simulation (--check = dry run)
ansible-playbook setup-webserver.yml -i inventory.ini --check

# Exécuter
ansible-playbook setup-webserver.yml -i inventory.ini

# Avec verbosité pour debugger
ansible-playbook setup-webserver.yml -i inventory.ini -vvv
```

---

## Les modules essentiels

=== "Fichiers & dossiers"
    ```yaml
    - name: Copier un fichier local vers le serveur
      copy:
        src: fichiers/nginx.conf     # Relatif au playbook
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: '0644'
        backup: true                  # Garde une copie de l'ancien fichier

    - name: Créer un dossier
      file:
        path: /var/www/monapp
        state: directory
        owner: www-data
        mode: '0755'

    - name: Créer un lien symbolique
      file:
        src: /etc/nginx/sites-available/monapp
        dest: /etc/nginx/sites-enabled/monapp
        state: link

    - name: Supprimer un fichier
      file:
        path: /tmp/fichier-temporaire.txt
        state: absent
    ```

=== "Paquets"
    ```yaml
    # Ubuntu/Debian
    - name: Installer des paquets
      apt:
        name:
          - nginx
          - postgresql
          - python3
        state: present          # present / absent / latest

    - name: Supprimer un paquet
      apt:
        name: apache2
        state: absent
        purge: true             # Supprime aussi les fichiers de config

    # CentOS/RHEL
    - name: Installer via yum
      yum:
        name: httpd
        state: latest

    # Agnostique (détecte automatiquement le gestionnaire)
    - name: Installer (multi-distrib)
      package:
        name: git
        state: present
    ```

=== "Services"
    ```yaml
    - name: Démarrer Nginx
      service:
        name: nginx
        state: started
        enabled: true       # Démarre au boot

    - name: Redémarrer PostgreSQL
      service:
        name: postgresql
        state: restarted

    - name: Recharger la config Nginx (sans downtime)
      service:
        name: nginx
        state: reloaded
    ```

=== "Commandes"
    ```yaml
    - name: Exécuter une commande simple
      command: /usr/bin/python3 --version
      register: python_version      # Stocker le résultat

    - name: Afficher la version
      debug:
        msg: "Python : {{ python_version.stdout }}"

    - name: Commande shell (pipes, redirections)
      shell: |
        cd /opt/app
        pip install -r requirements.txt 2>&1 | tee /tmp/pip.log

    - name: Seulement si le fichier n'existe pas
      command: /opt/setup.sh
      args:
        creates: /opt/.setup-done   # Ne s'exécute pas si ce fichier existe
    ```

---

## L'idempotence — Le super-pouvoir d'Ansible

!!! info "C'est quoi ?"
    Tu peux lancer le même playbook **10 fois** → résultat identique.
    Si Nginx est déjà installé, Ansible ne le réinstalle pas. Il vérifie avant d'agir.

```yaml
# ✅ Idempotent — vérifie l'état avant d'agir
- apt:
    name: nginx
    state: present      # "Il doit être présent" — pas "installe-le à chaque fois"

# ⚠️ Non idempotent — s'exécute à chaque fois
- command: apt-get install -y nginx
```

La sortie d'Ansible distingue :
- `ok` → Aucun changement (déjà dans l'état attendu)
- `changed` → Ansible a fait quelque chose
- `failed` → Erreur

---

## Ad-hoc commands — Sans playbook

```bash
# Tester la connectivité
ansible all -i inventory.ini -m ping

# Exécuter une commande sur tous les serveurs
ansible webservers -i inventory.ini -m command -a "uptime"

# Installer un paquet sur un groupe
ansible webservers -i inventory.ini -m apt -a "name=htop state=present" --become

# Copier un fichier
ansible all -i inventory.ini -m copy -a "src=./fichier.txt dest=/tmp/fichier.txt"

# Redémarrer un service
ansible webservers -i inventory.ini -m service -a "name=nginx state=restarted" --become

# Récupérer des facts (infos système)
ansible web1.example.com -i inventory.ini -m setup
ansible web1.example.com -i inventory.ini -m setup -a "filter=ansible_memory_mb"
```

---

## ansible.cfg — Configuration

```ini
# ansible.cfg (dans le dossier du projet)
[defaults]
inventory       = inventory.ini
remote_user     = ubuntu
private_key_file = ~/.ssh/id_rsa
host_key_checking = False
retry_files_enabled = False

[privilege_escalation]
become          = True
become_method   = sudo
become_user     = root
```

!!! success "Checkpoint débutant ✅"
    Tu sais : installer Ansible, créer un inventaire, écrire un playbook, utiliser les modules principaux et les ad-hoc commands.
