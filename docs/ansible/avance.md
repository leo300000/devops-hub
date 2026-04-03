# 🔴 Ansible — Avancé

## Ansible Vault — Chiffrer les secrets

```bash
# Chiffrer un fichier entier
ansible-vault encrypt group_vars/prod/secrets.yml

# Créer un fichier chiffré directement
ansible-vault create group_vars/prod/secrets.yml

# Voir/éditer un fichier chiffré
ansible-vault view secrets.yml
ansible-vault edit secrets.yml

# Lancer un playbook avec vault
ansible-playbook playbook.yml --ask-vault-pass
# ou avec un fichier de mot de passe
ansible-playbook playbook.yml --vault-password-file .vault_pass
```

Chiffrer seulement une valeur :
```bash
ansible-vault encrypt_string 'MonSuperMotDePasse' --name 'db_password'
# Sortie à coller dans ton YAML :
# db_password: !vault |
#   $ANSIBLE_VAULT;1.1;AES256
#   ...
```

---

## Dynamic Inventory — Inventaire depuis Azure/AWS

```python
# inventory_azure.py (ou utiliser le plugin officiel)
# plugin: azure_rm

# azure_rm.yml
plugin: azure.azcollection.azure_rm
auth_source: auto
include_vm_resource_groups:
  - rg-production

keyed_groups:
  - key: tags.role
    prefix: role
  - key: location
    prefix: location
```

```bash
# Utiliser l'inventaire dynamique
ansible-playbook playbook.yml -i azure_rm.yml

# Voir l'inventaire généré
ansible-inventory -i azure_rm.yml --list
```

---

## Molecule — Tester ses rôles

```bash
pip install molecule molecule-docker

# Initialiser les tests pour un rôle
cd roles/nginx
molecule init scenario

# Lancer les tests
molecule test  # create → converge → verify → destroy
```

```yaml
# molecule/default/converge.yml
- name: Converge
  hosts: all
  roles:
  - role: nginx

# molecule/default/verify.yml
- name: Verify
  hosts: all
  tasks:
  - name: Vérifier que Nginx répond
    uri:
      url: http://localhost
      status_code: 200
```

---

## Optimisation — Accélérer les playbooks

```ini
# ansible.cfg
[defaults]
forks = 20                    # Parallélisme (défaut: 5)
pipelining = True             # Réduit les connexions SSH
gathering = smart             # Ne collecte les facts qu'une fois

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s  # Réutilise les connexions SSH
```

```yaml
# Désactiver la collecte de facts si inutile
- hosts: webservers
  gather_facts: false          # Gain de temps si tu n'as pas besoin des facts
  tasks:
  - name: Redémarrer Nginx
    service:
      name: nginx
      state: restarted
```

---

## Stratégies d'exécution

```yaml
# Stratégie free — chaque hôte avance à son rythme (plus rapide)
- hosts: all
  strategy: free
  tasks: [...]

# Rolling update — màj progressive
- hosts: webservers
  serial: 2        # 2 serveurs à la fois
  # ou
  serial: "20%"    # 20% des serveurs à la fois
  tasks:
  - name: Mettre à jour l'app
    ...
```

---

## Ansible AWX / Tower — Interface Web

AWX (version open-source de Tower) offre :
- Interface web pour lancer les playbooks
- Gestion des credentials centralisée
- Scheduling des playbooks
- RBAC (qui peut lancer quoi)
- Audit trail complet

```bash
# Installer AWX avec Docker Compose
git clone https://github.com/ansible/awx.git
cd awx
make docker-compose-build
make docker-compose
```
