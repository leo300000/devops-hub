# 🚀 DevOps Hub

> Tutoriels DevOps de débutant à avancé — Terraform, Ansible, Azure, Kubernetes, Git, GitHub, CI/CD

## 🌐 Accéder au site

**👉 https://leo300000.github.io/devops-hub**

Le site est déployé automatiquement sur GitHub Pages à chaque push sur `master`.

---

## 📚 Contenu

| Techno | Débutant | Intermédiaire | Avancé |
|--------|:--------:|:-------------:|:------:|
| 🏗️ [Terraform](docs/terraform/) | ✅ | ✅ | ✅ |
| ⚙️ [Ansible](docs/ansible/) | ✅ | ✅ | ✅ |
| ☁️ [Azure](docs/azure/) | ✅ | ✅ | ✅ |
| 🐳 [Kubernetes](docs/kubernetes/) | ✅ | ✅ | ✅ |
| 🌿 [Git](docs/git/) | ✅ | ✅ | ✅ |
| 🐙 [GitHub](docs/github/) | ✅ | ✅ | ✅ |
| 🔄 [CI/CD GitHub Actions](docs/cicd/) | ✅ | ✅ | ✅ |

---

## 🛠️ Lancer le site en local

```bash
# Installer MkDocs Material
pip install mkdocs-material

# Cloner le repo
git clone https://github.com/leo300000/devops-hub.git
cd devops-hub

# Lancer en local (hot-reload)
mkdocs serve

# Ouvrir http://127.0.0.1:8000
```

## 🏗️ Build et déploiement

Le déploiement est automatique via GitHub Actions (`.github/workflows/deploy.yml`).

```bash
# Build manuel (génère le dossier site/)
mkdocs build

# Déployer manuellement sur GitHub Pages
mkdocs gh-deploy
```

---

## 📖 Documentation officielle

- [Terraform Registry](https://registry.terraform.io/)
- [HashiCorp Learn](https://developer.hashicorp.com/terraform/tutorials)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Ansible Docs](https://docs.ansible.com/)
- [Azure Docs](https://learn.microsoft.com/azure/)
- [Kubernetes Docs](https://kubernetes.io/docs/)
- [GitHub Actions Docs](https://docs.github.com/actions)
