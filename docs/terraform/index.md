# 🏗️ Terraform

> **Infrastructure as Code — Dessine ton infra comme du code**

---

## L'analogie parfaite

!!! quote ""
    Terraform, c'est comme un **plan d'architecte**.
    
    Tu décris sur le papier la maison que tu veux (nb de pièces, taille, etc.) et Terraform la construit pour toi — automatiquement, reproductible, à l'identique à chaque fois.

Sans Terraform :
```
Clique ici → remplis ce formulaire → attends → clique là → configure à la main
→ recommence sur 10 serveurs → prie pour ne pas avoir fait d'erreur
```

Avec Terraform :
```hcl
resource "azurerm_virtual_machine" "web" {
  count = 10
  # ... 10 VMs identiques en une commande
}
```

---

## Terraform vs les alternatives

| Outil | Type | Points forts | Points faibles |
|-------|------|-------------|----------------|
| **Terraform** | Multi-cloud | Universel, grande communauté | State management complexe |
| **Pulumi** | Multi-cloud | Code Python/JS/Go | Moins mature |
| **ARM Templates** | Azure only | Natif Azure | Verbeux, Azure seulement |
| **Bicep** | Azure only | Syntax plus propre que ARM | Azure seulement |
| **CloudFormation** | AWS only | Natif AWS | AWS seulement |

**👉 Terraform = le couteau suisse** — fonctionne partout (AWS, Azure, GCP, on-prem).

---

## Les 3 commandes à retenir

```bash
terraform init    # 📦 Télécharge les plugins (comme npm install)
terraform plan    # 👀 Montre ce qui va changer (sans toucher à rien)
terraform apply   # 🚀 Applique les changements
```

---

## Niveaux disponibles

- [🟢 Débutant](debutant.md) — Premiers pas, HCL, première ressource
- [🟡 Intermédiaire](intermediaire.md) — Modules, variables, state
- [🔴 Avancé](avance.md) — Remote state, workspaces, CI/CD
