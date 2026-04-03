# 🟡 Ansible — Intermédiaire

!!! info "Documentation officielle"
    - [Rôles Ansible](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
    - [Ansible Galaxy — Rôles populaires](https://galaxy.ansible.com/ui/search/?keywords=nginx&order_by=-download_count)
    - [Jinja2 Templates](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_templating.html)
    - [Variables et priorités](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html)

---

## Rôles — Organiser son code

!!! quote "Analogie"
    Un rôle = une **recette de cuisine réutilisable**.
    Tu l'écris une fois pour "installer Nginx" et tu l'utilises dans tous tes projets.

### Créer un rôle

```bash
ansible-galaxy init roles/nginx
# Structure créée automatiquement :
# roles/nginx/
# ├── tasks/main.yml
# ├── handlers/main.yml
# ├── templates/
# ├── files/
# ├── vars/main.yml
# ├── defaults/main.yml
# ├── meta/main.yml
# └── README.md
```

### Exemple complet — Rôle Nginx

```yaml
# roles/nginx/defaults/main.yml (valeurs par défaut, priorité basse)
nginx_port: 80
nginx_worker_processes: auto
nginx_worker_connections: 1024
nginx_user: www-data
nginx_sites: {}
```

```yaml
# roles/nginx/tasks/main.yml
---
- name: Installer Nginx
  apt:
    name: nginx
    state: present
  notify: Reload Nginx

- name: Configurer Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    mode: '0644'
  notify: Reload Nginx

- name: Supprimer le site par défaut
  file:
    path: /etc/nginx/sites-enabled/default
    state: absent
  notify: Reload Nginx

- name: Configurer les virtual hosts
  template:
    src: vhost.conf.j2
    dest: "/etc/nginx/sites-available/{{ item.key }}"
  loop: "{{ nginx_sites | dict2items }}"
  notify: Reload Nginx

- name: Activer les virtual hosts
  file:
    src: "/etc/nginx/sites-available/{{ item.key }}"
    dest: "/etc/nginx/sites-enabled/{{ item.key }}"
    state: link
  loop: "{{ nginx_sites | dict2items }}"
  notify: Reload Nginx

- name: Démarrer et activer Nginx
  service:
    name: nginx
    state: started
    enabled: true
```

```yaml
# roles/nginx/handlers/main.yml
---
- name: Reload Nginx
  service:
    name: nginx
    state: reloaded

- name: Restart Nginx
  service:
    name: nginx
    state: restarted
```

```ini
# roles/nginx/templates/nginx.conf.j2
user {{ nginx_user }};
worker_processes {{ nginx_worker_processes }};
pid /run/nginx.pid;

events {
    worker_connections {{ nginx_worker_connections }};
}

http {
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logs
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # Gzip
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;

    include /etc/nginx/sites-enabled/*;
}
```

---

## Ansible Galaxy — Utiliser des rôles de la communauté

```bash
# Chercher des rôles
ansible-galaxy search nginx --author geerlingguy

# Installer un rôle (geerlingguy est le plus connu)
ansible-galaxy install geerlingguy.nginx
ansible-galaxy install geerlingguy.postgresql
ansible-galaxy install geerlingguy.docker

# Voir les rôles installés
ansible-galaxy list

# requirements.yml — Gérer les dépendances de rôles
```

```yaml
# requirements.yml
---
roles:
  - name: geerlingguy.nginx
    version: "3.2.0"
  - name: geerlingguy.postgresql
    version: "3.4.1"
  - src: https://github.com/geerlingguy/ansible-role-docker
    name: geerlingguy.docker
    version: "7.1.0"

collections:
  - name: community.postgresql
    version: "3.4.0"
  - name: azure.azcollection
    version: "1.19.0"
```

```bash
# Installer depuis requirements.yml
ansible-galaxy install -r requirements.yml
ansible-galaxy collection install -r requirements.yml
```

### Utiliser les rôles communautaires

```yaml
# playbook.yml
---
- hosts: webservers
  become: true
  vars:
    nginx_vhosts:
      - listen: "80"
        server_name: "monapp.example.com"
        root: "/var/www/monapp"
        index: "index.html"
        extra_parameters: |
          location / {
            try_files $uri $uri/ =404;
          }

  roles:
    - geerlingguy.nginx

- hosts: databases
  become: true
  vars:
    postgresql_version: "15"
    postgresql_databases:
      - name: monapp
    postgresql_users:
      - name: monapp
        password: "{{ db_password }}"

  roles:
    - geerlingguy.postgresql
```

---

## Templates Jinja2 — Fichiers dynamiques

```ini
# templates/app.conf.j2
[database]
host = {{ db_host }}
port = {{ db_port | default(5432) }}
name = {{ db_name }}
user = {{ db_user }}

[server]
host = {{ ansible_default_ipv4.address }}   {# Variable de fact automatique #}
workers = {{ ansible_processor_vcpus * 2 }} {# Calcul dynamique #}
debug = {{ 'true' if environment == 'dev' else 'false' }}

[cache]
{% for server in cache_servers %}
server = {{ server }}
{% endfor %}
```

```yaml
# Utiliser le template
- name: Configurer l'application
  template:
    src: app.conf.j2
    dest: /etc/monapp/app.conf
    owner: monapp
    mode: '0640'
  vars:
    db_host: "{{ hostvars['db1.example.com']['ansible_default_ipv4']['address'] }}"
    db_name: monapp
    db_user: monapp
    cache_servers:
      - "redis1:6379"
      - "redis2:6379"
```

---

## Variables et priorité

```
Plus haute priorité
│  1. Extra vars: -e "var=value"
│  2. Task vars
│  3. Block vars
│  4. Registered vars
│  5. Set_fact
│  6. Playbook vars
│  7. Host vars (host_vars/hostname.yml)
│  8. Group vars (group_vars/groupname.yml)
│  9. Role vars (vars/main.yml)
│ 10. Role defaults (defaults/main.yml)
Plus basse priorité
```

```
├── group_vars/
│   ├── all.yml          # S'applique à tous les hosts
│   ├── webservers.yml   # S'applique au groupe webservers
│   └── databases.yml
└── host_vars/
    ├── web1.example.com.yml   # S'applique uniquement à ce host
    └── db1.example.com.yml
```

```yaml
# group_vars/all.yml
environment: prod
ntp_servers:
  - 0.fr.pool.ntp.org
  - 1.fr.pool.ntp.org

# group_vars/webservers.yml
nginx_port: 443
ssl_enabled: true

# host_vars/web1.example.com.yml
nginx_port: 8080  # Surcharge pour ce serveur uniquement
```

---

## Conditions et boucles avancées

```yaml
# Conditions
- name: Installer le bon package selon l'OS
  package:
    name: "{{ 'nginx' if ansible_os_family == 'Debian' else 'httpd' }}"
    state: present

- name: Configurer seulement si prod
  template:
    src: prod.conf.j2
    dest: /etc/app/app.conf
  when:
    - environment == "prod"
    - ansible_memory_mb.real.total > 4096

# Boucle avec liste
- name: Créer des utilisateurs
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
    shell: /bin/bash
  loop:
    - { name: alice, groups: "sudo,docker" }
    - { name: bob,   groups: "docker" }

# Boucle avec dict
- name: Créer des dossiers par app
  file:
    path: "/var/www/{{ item.key }}"
    state: directory
    owner: "{{ item.value.owner }}"
  loop: "{{ apps | dict2items }}"
  vars:
    apps:
      frontend: { owner: www-data }
      api:      { owner: appuser }

# Boucle with_items + conditions
- name: Copier les configs selon l'environnement
  copy:
    src: "configs/{{ environment }}/{{ item }}"
    dest: "/etc/app/{{ item }}"
  loop:
    - database.conf
    - cache.conf
    - logging.conf
  when: environment in ['staging', 'prod']
```

---

## Tags — Exécuter une partie du playbook

```yaml
- name: Installer Nginx
  apt:
    name: nginx
  tags:
    - nginx
    - install

- name: Configurer Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  tags:
    - nginx
    - config

- name: Déployer l'application
  git:
    repo: https://github.com/user/app.git
    dest: /var/www/app
  tags:
    - app
    - deploy
```

```bash
# Lancer uniquement la config Nginx
ansible-playbook playbook.yml --tags nginx,config

# Tout sauf le déploiement
ansible-playbook playbook.yml --skip-tags deploy

# Lister les tags disponibles
ansible-playbook playbook.yml --list-tags
```

!!! success "Checkpoint intermédiaire ✅"
    Tu maîtrises : rôles, Ansible Galaxy, templates Jinja2, variables/priorités, conditions, boucles, tags.
