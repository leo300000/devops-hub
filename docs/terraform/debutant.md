# 🟢 Terraform — Débutant

## La syntaxe HCL en 5 minutes

HCL (HashiCorp Configuration Language) ressemble à du JSON, mais en lisible.

```hcl
# Un bloc resource = une ressource cloud
resource "type_de_ressource" "nom_local" {
  parametre = "valeur"
}
```

!!! tip "Règle d'or"
    `"type_de_ressource"` = ce que tu crées (VM, réseau, base de données...)
    `"nom_local"` = le nom que TU choisis pour y faire référence dans ton code

---

## Ton premier fichier Terraform

Crée un fichier `main.tf` :

```hcl
# 1. Dis à Terraform quel cloud utiliser
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

# 2. Configure le provider
provider "azurerm" {
  features {}
}

# 3. Crée un resource group
resource "azurerm_resource_group" "mon_projet" {
  name     = "rg-mon-premier-projet"
  location = "France Central"
}
```

---

## Les 3 commandes du quotidien

```bash
# Étape 1 — Initialise le projet (une seule fois)
terraform init
# ✅ Télécharge le provider azurerm

# Étape 2 — Prévisualise les changements
terraform plan
# ✅ Affiche ce qui va être créé/modifié/détruit
# ❌ Ne touche à RIEN encore

# Étape 3 — Applique
terraform apply
# Terraform te demande de taper "yes" pour confirmer
```

!!! warning "Ne jamais oublier le `plan`"
    Toujours faire un `terraform plan` avant un `apply`. C'est comme relire avant d'envoyer un email important.

---

## Le fichier state (`terraform.tfstate`)

Terraform garde en mémoire ce qu'il a créé dans un fichier `terraform.tfstate`.

```
ton code → terraform → azure
                 ↓
           terraform.tfstate  (la "mémoire" de Terraform)
```

!!! danger "Ne jamais éditer tfstate à la main"
    C'est comme modifier directement la base de données — tu vas tout casser.

---

## Variables — Rendre ton code réutilisable

```hcl
# variables.tf
variable "environment" {
  description = "Nom de l'environnement"
  type        = string
  default     = "dev"
}

variable "location" {
  description = "Région Azure"
  type        = string
  default     = "France Central"
}

# main.tf — utiliser les variables
resource "azurerm_resource_group" "rg" {
  name     = "rg-${var.environment}"
  location = var.location
}
```

Passer une variable en ligne de commande :
```bash
terraform apply -var="environment=prod"
```

---

## Outputs — Récupérer des infos après création

```hcl
# outputs.tf
output "resource_group_id" {
  description = "L'ID du resource group créé"
  value       = azurerm_resource_group.rg.id
}
```

```bash
terraform output resource_group_id
# /subscriptions/xxx/resourceGroups/rg-dev
```

---

## Détruire ce qu'on a créé

```bash
terraform destroy
# ⚠️ Supprime TOUT ce que Terraform a créé
# Utile pour les environnements de test
```

---

## Récap — Les fichiers importants

| Fichier | Rôle |
|---------|------|
| `main.tf` | Ressources principales |
| `variables.tf` | Déclaration des variables |
| `outputs.tf` | Valeurs à afficher après apply |
| `terraform.tfvars` | Valeurs des variables (ne pas commiter si secrets !) |
| `terraform.tfstate` | État actuel (ne pas éditer manuellement) |

!!! success "Checkpoint débutant ✅"
    Tu sais : init / plan / apply / destroy, créer un resource group, utiliser des variables et des outputs. Passe au niveau intermédiaire !
