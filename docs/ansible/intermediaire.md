# 🟡 Ansible — Intermédiaire

## Rôles — Organiser son code

Un rôle = une structure standardisée pour regrouper des tâches.

```
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml        # Les tâches
    ├── handlers/
    │   └── main.yml        # Les handlers
    ├── templates/
    │   └── nginx.conf.j2   # Templates Jinja2
    ├── files/
    │   └── index.html      # Fichiers statiques
    ├── vars/
    │   └── main.yml        # Variables du rôle
    └── defaults/
        └── main.yml        # Variables par défaut (priorité basse)
```

Utiliser un rôle :
```yaml
# playbook.yml
- hosts: webservers
  roles:
  - nginx
  - { role: database, db_name: "mon_app" }
```

---

## Handlers — Réagir aux changements

```yaml
# tasks/main.yml
- name: Copier la config Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Reload Nginx            # Déclenche le handler si changement

# handlers/main.yml
- name: Reload Nginx
  service:
    name: nginx
    state: reloaded
```

!!! tip "Les handlers ne s'exécutent qu'une seule fois"
    Même si 5 tâches notifient "Reload Nginx", il ne se recharge qu'une seule fois à la fin du play.

---

## Templates Jinja2

```ini
# templates/nginx.conf.j2
server {
    listen {{ nginx_port | default(80) }};
    server_name {{ ansible_hostname }};

    location / {
        root {{ app_root }};
        {% if enable_gzip %}
        gzip on;
        {% endif %}
    }
}
```

```yaml
# vars/main.yml
nginx_port: 443
app_root: /var/www/html
enable_gzip: true
```

---

## Variables — Priorité (du plus fort au plus faible)

```
1. Extra vars (-e)         ansible-playbook -e "env=prod"
2. Task vars               vars: dans une task
3. Playbook vars           vars: dans le play
4. Host vars               host_vars/serveur.yml
5. Group vars              group_vars/webservers.yml
6. Role vars               roles/nginx/vars/main.yml
7. Role defaults           roles/nginx/defaults/main.yml
```

```yaml
# group_vars/webservers.yml
nginx_port: 80
app_env: staging

# host_vars/web1.example.com.yml
nginx_port: 443          # Surcharge pour web1 uniquement
```

---

## Conditions et boucles

=== "Conditions"
    ```yaml
    - name: Installer sur Debian seulement
      apt:
        name: nginx
      when: ansible_os_family == "Debian"

    - name: Installer sur RedHat seulement
      yum:
        name: httpd
      when: ansible_os_family == "RedHat"
    ```

=== "Boucles"
    ```yaml
    - name: Créer plusieurs utilisateurs
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"
        state: present
      loop:
        - { name: alice, groups: sudo }
        - { name: bob, groups: docker }
        - { name: charlie, groups: www-data }
    ```

---

## Tags — Exécuter une partie du playbook

```yaml
- name: Installer Nginx
  apt:
    name: nginx
  tags: [nginx, install]

- name: Configurer Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  tags: [nginx, config]
```

```bash
# Exécuter seulement les tâches taguées "config"
ansible-playbook playbook.yml --tags config

# Exclure les tâches taguées "install"
ansible-playbook playbook.yml --skip-tags install
```

!!! success "Checkpoint intermédiaire ✅"
    Tu maîtrises : rôles, handlers, templates Jinja2, variables, conditions, boucles, tags.
