# 🟢 Ansible — Débutant

## L'inventaire — Tes serveurs

```ini
# inventory.ini
[webservers]
web1.example.com
web2.example.com
192.168.1.10

[databases]
db1.example.com ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

---

## Ton premier playbook

```yaml
# deploy-nginx.yml
---
- name: Installer et configurer Nginx
  hosts: webservers          # Groupe de l'inventaire
  become: true               # Sudo

  tasks:
  - name: Mettre à jour les paquets
    apt:
      update_cache: true
      cache_valid_time: 3600

  - name: Installer Nginx
    apt:
      name: nginx
      state: present         # present = installer, absent = désinstaller

  - name: Démarrer Nginx
    service:
      name: nginx
      state: started
      enabled: true          # Démarrer au boot
```

```bash
# Exécuter le playbook
ansible-playbook deploy-nginx.yml -i inventory.ini

# Test de connexion
ansible all -i inventory.ini -m ping
```

---

## Les modules essentiels

=== "Fichiers"
    ```yaml
    - name: Copier un fichier
      copy:
        src: fichier-local.conf
        dest: /etc/nginx/nginx.conf
        owner: root
        mode: '0644'

    - name: Créer un dossier
      file:
        path: /var/www/monapp
        state: directory
        owner: www-data
        mode: '0755'
    ```

=== "Paquets"
    ```yaml
    # Ubuntu/Debian
    - apt:
        name: [nginx, curl, git]
        state: present

    # CentOS/RHEL
    - yum:
        name: httpd
        state: latest
    ```

=== "Commandes"
    ```yaml
    - name: Lancer un script
      command: /opt/scripts/setup.sh

    - name: Commande shell complexe
      shell: |
        cd /opt/app
        pip install -r requirements.txt

    - name: Récupérer la sortie
      command: whoami
      register: resultat

    - debug:
        msg: "Utilisateur : {{ resultat.stdout }}"
    ```

---

## L'idempotence — Le super-pouvoir d'Ansible

!!! info "C'est quoi l'idempotence ?"
    Tu peux lancer le même playbook **10 fois** → le résultat sera identique.
    Si Nginx est déjà installé, Ansible ne le réinstalle pas.

```yaml
# ✅ Idempotent — Ansible vérifie avant d'agir
- apt:
    name: nginx
    state: present   # "Il doit être présent" → pas "installe-le"

# ⚠️ Pas idempotent — s'exécute à chaque fois
- command: apt-get install nginx
```

---

## Ad-hoc commands — Sans playbook

```bash
# Ping tous les serveurs
ansible all -i inventory.ini -m ping

# Créer un fichier
ansible webservers -i inventory.ini -m file -a "path=/tmp/test state=touch"

# Redémarrer un service
ansible webservers -i inventory.ini -m service -a "name=nginx state=restarted" --become

# Exécuter une commande
ansible all -i inventory.ini -m command -a "uptime"
```

!!! success "Checkpoint débutant ✅"
    Tu sais créer un inventaire, écrire un playbook et utiliser les modules principaux.
