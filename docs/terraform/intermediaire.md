# 🟡 Terraform — Intermédiaire

## Modules — Le principe DRY

!!! quote "Analogie"
    Un module Terraform = une **fonction réutilisable**.
    Tu l'écris une fois, tu l'appelles partout.

Structure d'un module :
```
modules/
└── virtual_machine/
    ├── main.tf       # Les ressources
    ├── variables.tf  # Les inputs
    └── outputs.tf    # Les outputs
```

Appeler un module :
```hcl
# main.tf
module "vm_web" {
  source = "./modules/virtual_machine"

  name           = "vm-web"
  resource_group = azurerm_resource_group.rg.name
  size           = "Standard_B2s"
}

module "vm_api" {
  source = "./modules/virtual_machine"  # Même module !

  name           = "vm-api"
  resource_group = azurerm_resource_group.rg.name
  size           = "Standard_D4s_v3"
}
```

---

## Remote State — Partager l'état en équipe

Par défaut, le state est **local** (ton PC). En équipe, il doit être **partagé**.

```hcl
# backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "satfstate12345"
    container_name       = "tfstate"
    key                  = "prod/terraform.tfstate"
  }
}
```

!!! info "Pourquoi le remote state ?"
    - Partage entre plusieurs personnes
    - Verrou automatique (deux personnes ne peuvent pas apply en même temps)
    - Sauvegarde automatique

---

## Workspaces — Multi-environnements

```bash
# Créer et switcher d'environnement
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

terraform workspace list
# * dev
#   staging
#   prod

terraform workspace select prod
```

Utiliser le workspace dans le code :
```hcl
resource "azurerm_resource_group" "rg" {
  name     = "rg-${terraform.workspace}"  # rg-dev, rg-prod, etc.
  location = var.location
}
```

---

## `count` et `for_each` — Créer plusieurs ressources

=== "count — Nombre fixe"
    ```hcl
    resource "azurerm_virtual_machine" "web" {
      count = 3  # Crée 3 VMs identiques

      name = "vm-web-${count.index}"  # vm-web-0, vm-web-1, vm-web-2
    }
    ```

=== "for_each — Basé sur une map"
    ```hcl
    variable "vms" {
      default = {
        "web"  = "Standard_B2s"
        "api"  = "Standard_D4s_v3"
        "db"   = "Standard_E4s_v3"
      }
    }

    resource "azurerm_virtual_machine" "servers" {
      for_each = var.vms

      name = "vm-${each.key}"
      size = each.value
    }
    ```

!!! tip "`for_each` > `count`"
    Préfère `for_each` — si tu supprimes un élément du milieu avec `count`, Terraform recrée toutes les ressources après. Avec `for_each`, il supprime uniquement celle concernée.

---

## Data Sources — Récupérer des ressources existantes

```hcl
# Récupérer un resource group existant (pas créé par Terraform)
data "azurerm_resource_group" "existing" {
  name = "rg-existant-dans-azure"
}

# L'utiliser dans une ressource
resource "azurerm_virtual_network" "vnet" {
  resource_group_name = data.azurerm_resource_group.existing.name
  location            = data.azurerm_resource_group.existing.location
}
```

---

## `locals` — Variables calculées

```hcl
locals {
  # Nom standardisé pour toutes les ressources
  prefix = "${var.project}-${var.environment}"
  
  tags = {
    Environment = var.environment
    Project     = var.project
    ManagedBy   = "Terraform"
  }
}

resource "azurerm_resource_group" "rg" {
  name = "rg-${local.prefix}"
  tags = local.tags
}
```

---

## Commandes utiles

```bash
terraform fmt           # Formate le code automatiquement
terraform validate      # Vérifie la syntaxe sans se connecter au cloud
terraform state list    # Liste toutes les ressources dans le state
terraform state show azurerm_resource_group.rg  # Détails d'une ressource
terraform import azurerm_resource_group.rg /subscriptions/.../rg-existant  # Importer une ressource existante
```

!!! success "Checkpoint intermédiaire ✅"
    Tu sais : modules, remote state, workspaces, for_each, data sources, locals. Prêt pour le niveau avancé !
