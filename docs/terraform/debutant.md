# 🟢 Terraform — Débutant

!!! info "Documentation officielle"
    - [Terraform Docs](https://developer.hashicorp.com/terraform/docs)
    - [AzureRM Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
    - [HashiCorp Learn — Azure](https://developer.hashicorp.com/terraform/tutorials/azure-get-started)

---

## La syntaxe HCL en 5 minutes

HCL (HashiCorp Configuration Language) ressemble à du JSON, mais humainement lisible.

```hcl
# Un bloc resource = une ressource cloud
resource "type_de_ressource" "nom_local" {
  parametre = "valeur"
}
```

!!! tip "Règle d'or"
    `"type_de_ressource"` → ce que tu crées (`azurerm_resource_group`, `azurerm_virtual_network`...)
    `"nom_local"` → le nom que **toi** tu choisis pour y faire référence dans le code

---

## Installation

```bash
# Windows (winget)
winget install Hashicorp.Terraform

# Mac
brew install terraform

# Vérifier
terraform version
# Terraform v1.7.0
```

---

## Configurer l'accès à Azure

```bash
# 1. Installer Azure CLI
winget install Microsoft.AzureCLI

# 2. Se connecter
az login

# 3. Sélectionner ta subscription
az account list --output table
az account set --subscription "Mon-Abonnement"

# 4. Terraform utilisera automatiquement tes credentials Azure CLI
```

---

## Ton premier fichier Terraform avec AzureRM

```hcl
# main.tf

terraform {
  required_version = ">= 1.5.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.85"
    }
  }
}

provider "azurerm" {
  features {}
  # Terraform utilise automatiquement az login
  # En CI/CD, on utilisera des variables d'environnement
}

# Resource Group — le "dossier" Azure
resource "azurerm_resource_group" "rg" {
  name     = "rg-mon-premier-projet"
  location = "France Central"

  tags = {
    Environment = "dev"
    ManagedBy   = "Terraform"
  }
}
```

---

## Les 3 commandes du quotidien

```bash
# 1. Initialise le projet (télécharge le provider azurerm)
terraform init

# Output :
# Initializing provider plugins...
# - Installing hashicorp/azurerm v3.85.0...
# ✅ Terraform initialized successfully!

# 2. Prévisualise les changements
terraform plan

# Output :
# Terraform will perform the following actions:
#   + azurerm_resource_group.rg will be created
#     + name     = "rg-mon-premier-projet"
#     + location = "francecentral"
# Plan: 1 to add, 0 to change, 0 to destroy.

# 3. Applique
terraform apply
# Tape "yes" pour confirmer
```

!!! warning "Toujours faire `plan` avant `apply`"
    Le plan est comme relire avant d'envoyer un email important. Il montre exactement ce qui va changer **sans toucher à rien**.

---

## Créer un Storage Account

```hcl
resource "azurerm_storage_account" "storage" {
  name                     = "stmonprojetdev001"  # Doit être unique globalement
  resource_group_name      = azurerm_resource_group.rg.name      # Référence le RG créé au-dessus
  location                 = azurerm_resource_group.rg.location  # Même région
  account_tier             = "Standard"
  account_replication_type = "LRS"  # Locally Redundant Storage

  tags = {
    Environment = "dev"
  }
}
```

!!! info "Référencement entre ressources"
    `azurerm_resource_group.rg.name` = type + nom_local + attribut
    Terraform crée les ressources dans le bon ordre automatiquement.

---

## Variables — Rendre ton code réutilisable

```hcl
# variables.tf
variable "environment" {
  description = "Nom de l'environnement (dev, staging, prod)"
  type        = string
  default     = "dev"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "L'environnement doit être dev, staging ou prod."
  }
}

variable "location" {
  description = "Région Azure"
  type        = string
  default     = "France Central"
}

variable "project_name" {
  description = "Nom court du projet (utilisé dans le nommage)"
  type        = string
}
```

```hcl
# main.tf — utiliser les variables
resource "azurerm_resource_group" "rg" {
  name     = "rg-${var.project_name}-${var.environment}"
  location = var.location
}
```

```hcl
# terraform.tfvars — valeurs des variables
project_name = "monapp"
environment  = "dev"
location     = "France Central"
```

```bash
# Passer une variable en CLI
terraform apply -var="environment=prod"

# Utiliser un fichier de variables différent
terraform apply -var-file="prod.tfvars"
```

---

## Outputs — Récupérer des infos après création

```hcl
# outputs.tf
output "resource_group_name" {
  description = "Nom du resource group créé"
  value       = azurerm_resource_group.rg.name
}

output "storage_account_connection_string" {
  description = "Chaîne de connexion du storage account"
  value       = azurerm_storage_account.storage.primary_connection_string
  sensitive   = true  # N'apparaît pas dans les logs
}
```

```bash
# Voir les outputs après apply
terraform output
terraform output resource_group_name

# Récupérer un output sensitif
terraform output -raw storage_account_connection_string
```

---

## Le fichier state (`terraform.tfstate`)

```
ton code HCL → terraform apply → Azure
                      ↓
               terraform.tfstate   ← mémoire de ce qui a été créé
```

| Fichier | Rôle | Committer ? |
|---------|------|-------------|
| `terraform.tfstate` | État actuel | ❌ Non (contient des secrets) |
| `terraform.tfstate.backup` | Sauvegarde | ❌ Non |
| `.terraform/` | Cache providers | ❌ Non |
| `*.tfvars` | Variables | ⚠️ Seulement si pas de secrets |

**→ Ajoute `.terraform/`, `*.tfstate*` à ton `.gitignore` !**

---

## Détruire l'infrastructure

```bash
terraform destroy
# Montre ce qui va être supprimé
# Tape "yes" pour confirmer

# Ou détruire une ressource spécifique
terraform destroy -target=azurerm_storage_account.storage
```

!!! danger "Ne jamais `destroy` en prod sans backup"
    `destroy` supprime tout. En production, il faut des backups et une procédure validée.

---

## Récap — Les fichiers d'un projet Terraform

```
mon-projet/
├── main.tf           # Ressources principales
├── variables.tf      # Déclaration des variables
├── outputs.tf        # Valeurs à afficher après apply
├── providers.tf      # Configuration des providers
├── terraform.tfvars  # Valeurs des variables (ne pas commiter si secrets)
└── .gitignore        # Exclure .terraform/, *.tfstate*
```

!!! success "Checkpoint débutant ✅"
    Tu sais : configurer Azure, init/plan/apply/destroy, créer un RG et un Storage Account, utiliser variables et outputs.
