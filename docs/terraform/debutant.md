# 🟢 Terraform — Débutant

!!! info "Documentation officielle"
    - [Terraform Docs](https://developer.hashicorp.com/terraform/docs)
    - [AzureRM Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
    - [HashiCorp Learn — Azure](https://developer.hashicorp.com/terraform/tutorials/azure-get-started)

---

## C'est quoi Terraform ?

Terraform est un outil d'**Infrastructure as Code (IaC)**. Au lieu de créer des serveurs, réseaux et bases de données à la main via des clics dans une interface, tu les décris dans des fichiers texte, et Terraform les crée automatiquement.

!!! quote "Analogie"
    Terraform, c'est comme un **plan d'architecte**.
    Le plan décrit exactement la maison (nb de pièces, dimensions, matériaux). L'entrepreneur (Terraform) construit exactement ce qui est décrit — toujours pareil, sans improvisation.

**Sans Terraform :**
```
Clique ici → remplis ce formulaire → attends → configure à la main
→ recommence sur 10 serveurs → prie pour ne pas avoir fait d'erreur
→ impossible à reproduire exactement
```

**Avec Terraform :**
```hcl
resource "azurerm_virtual_machine" "web" {
  count = 10  # 10 VMs identiques en une seule commande
  # ...
}
```

---

## C'est quoi un Provider ?

**C'est quoi ?** Un Provider est le **plugin** qui permet à Terraform de communiquer avec un service cloud. Il y a un provider pour Azure (`azurerm`), AWS (`aws`), GCP (`google`), Kubernetes, GitHub, et des centaines d'autres.

!!! quote "Analogie"
    Le Provider = le **traducteur**.
    Tu écris en HCL "crée-moi un serveur", le provider Azure traduit ça en appels API Azure.

```hcl
# On déclare quels providers on utilise et leur version
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"  # Où télécharger le provider
      version = "~> 3.85"            # Version à utiliser (~> = compatible avec 3.85.x)
    }
  }
}

# On configure le provider
provider "azurerm" {
  features {}  # Obligatoire pour azurerm, même vide
  # Terraform utilise automatiquement tes credentials `az login`
}
```

---

## C'est quoi une Resource ?

**C'est quoi ?** Une Resource est l'unité de base de Terraform. Chaque resource représente **un objet dans le cloud** : un serveur, un réseau, une base de données, un bucket de stockage...

```hcl
resource "TYPE" "NOM_LOCAL" {
  # Paramètres de la ressource
}
```

- `TYPE` → ce que tu crées (`azurerm_resource_group`, `azurerm_virtual_network`...)
- `NOM_LOCAL` → le nom que **toi** tu choisis pour y faire référence dans le reste du code

!!! tip "Nomenclature"
    `azurerm_resource_group` → `azurerm` = le provider, `resource_group` = le type d'objet Azure
    Tu trouves tous les types dans la [doc du provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs).

---

## C'est quoi un Resource Group Azure ?

**C'est quoi ?** Un Resource Group est un **dossier logique** dans Azure qui regroupe des ressources liées. C'est obligatoire — toute ressource Azure doit appartenir à un Resource Group.

```hcl
resource "azurerm_resource_group" "rg" {
  # "rg" = le nom local, on peut l'appeler comme on veut
  # On y fera référence avec : azurerm_resource_group.rg.name
  
  name     = "rg-mon-premier-projet"  # Le nom affiché dans Azure
  location = "France Central"          # La région Azure où créer le RG
  
  tags = {                             # Étiquettes pour organiser et filtrer
    Environment = "dev"
    ManagedBy   = "Terraform"
  }
}
```

---

## Installation

```bash
# Windows
winget install Hashicorp.Terraform

# Mac
brew install terraform

# Vérifier
terraform version
```

## Configurer l'accès à Azure

```bash
# Installer Azure CLI
winget install Microsoft.AzureCLI

# Se connecter (ouvre le navigateur)
az login

# Vérifier qu'on est sur la bonne subscription
az account show

# Terraform utilisera automatiquement ces credentials
```

---

## Les 3 commandes du quotidien

**`terraform init`** — C'est quoi ? Initialise le projet. Télécharge les providers déclarés dans le code. À faire une seule fois au démarrage, et à refaire si tu changes de provider ou de version.

```bash
terraform init
# ✅ Télécharge le provider azurerm
# ✅ Configure le backend (remote state)
# ✅ Crée le dossier .terraform/
```

---

**`terraform plan`** — C'est quoi ? Compare ton code avec l'état actuel de l'infrastructure et affiche exactement ce qui va changer. **Ne touche à rien** — c'est une prévisualisation.

```bash
terraform plan

# Output :
# Terraform will perform the following actions:
#
#   # azurerm_resource_group.rg will be created
#   + resource "azurerm_resource_group" "rg" {
#       + id       = (known after apply)
#       + location = "francecentral"
#       + name     = "rg-mon-premier-projet"
#     }
#
# Plan: 1 to add, 0 to change, 0 to destroy.
```

Les symboles signifient :
- `+` → sera **créé**
- `~` → sera **modifié**
- `-` → sera **détruit**
- `-/+` → sera **détruit puis recréé**

!!! warning "Toujours faire `plan` avant `apply`"
    Le plan, c'est comme relire un email avant d'appuyer sur Envoyer. Repère les `destroy` inattendus !

---

**`terraform apply`** — C'est quoi ? Applique les changements — crée, modifie ou détruit les ressources dans Azure.

```bash
terraform apply
# Terraform affiche le plan, puis demande confirmation :
# Do you want to perform these actions? (yes/no): yes
#
# azurerm_resource_group.rg: Creating...
# azurerm_resource_group.rg: Creation complete after 3s
#
# Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

---

## C'est quoi le State (`terraform.tfstate`) ?

**C'est quoi ?** Le state est la **mémoire** de Terraform. Il enregistre ce qu'il a créé dans Azure. Sans state, Terraform ne saurait pas ce qui existe déjà et recréerait tout à chaque `apply`.

```
ton code HCL
      ↓
 terraform apply
      ↓
   Azure (création)
      ↓
terraform.tfstate  ← "J'ai créé rg-mon-projet avec l'ID /subscriptions/.../rg-mon-projet"
```

```bash
# Voir ce que Terraform gère
terraform state list
# azurerm_resource_group.rg
# azurerm_storage_account.storage

# Détails d'une ressource
terraform state show azurerm_resource_group.rg
```

!!! danger "3 règles absolues sur le state"
    1. Ne jamais l'éditer à la main
    2. Ne jamais le commiter dans Git (contient des données sensibles)
    3. En équipe, le stocker dans Azure Blob Storage (remote state)

---

## Créer un Storage Account — Exemple complet

```hcl
# Un Storage Account = stockage de fichiers dans Azure (équivalent S3 d'AWS)

resource "azurerm_storage_account" "storage" {
  # Le nom doit être : unique globalement, 3-24 chars, minuscules et chiffres uniquement
  name = "stmonprojetdev001"

  # On référence le resource group créé plus haut
  # Syntaxe : TYPE.NOM_LOCAL.ATTRIBUT
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location  # Même région que le RG

  account_tier             = "Standard"  # Standard ou Premium
  account_replication_type = "LRS"       # LRS = stockage local (moins cher, pour dev)

  tags = {
    Environment = "dev"
  }
}
```

!!! info "Référencement entre ressources"
    `azurerm_resource_group.rg.name` = le nom du resource group créé par Terraform.
    Terraform comprend la dépendance et crée le RG **avant** le Storage Account, automatiquement.

---

## Variables — Paramétrer ton code

**C'est quoi ?** Les variables permettent de rendre ton code réutilisable. Au lieu de mettre `"dev"` en dur partout, tu mets `var.environment` et tu changes la valeur une seule fois.

```hcl
# variables.tf — Déclare les variables disponibles
variable "environment" {
  description = "Nom de l'environnement"
  type        = string
  default     = "dev"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Doit être dev, staging ou prod."
  }
}

variable "location" {
  description = "Région Azure"
  type        = string
  default     = "France Central"
}

variable "project_name" {
  description = "Nom court du projet"
  type        = string
  # Pas de default → Terraform le demandera interactivement
}
```

```hcl
# main.tf — Utilise les variables avec var.NOM
resource "azurerm_resource_group" "rg" {
  name     = "rg-${var.project_name}-${var.environment}"
  location = var.location
}
```

```hcl
# terraform.tfvars — Donne les valeurs (ne pas commiter si contient des secrets)
project_name = "monapp"
environment  = "dev"
```

```bash
# Passer une variable en CLI (priorité la plus haute)
terraform apply -var="environment=prod"

# Utiliser un fichier de variables différent
terraform apply -var-file="prod.tfvars"
```

---

## Outputs — Récupérer des informations

**C'est quoi ?** Les outputs affichent des valeurs après un `apply`. Utile pour récupérer des IDs, IPs, URLs générés par Azure.

```hcl
# outputs.tf
output "resource_group_name" {
  description = "Nom du resource group"
  value       = azurerm_resource_group.rg.name
}

output "storage_connection_string" {
  description = "Chaîne de connexion du Storage Account"
  value       = azurerm_storage_account.storage.primary_connection_string
  sensitive   = true  # Masqué dans les logs, affiché seulement si demandé explicitement
}
```

```bash
terraform output                                    # Affiche tous les outputs
terraform output resource_group_name               # Un output spécifique
terraform output -raw storage_connection_string    # Sans les guillemets (pour des scripts)
```

---

## Supprimer l'infrastructure

```bash
# Détruire TOUT ce que Terraform gère
terraform destroy
# Demande confirmation : yes

# Détruire une seule ressource
terraform destroy -target=azurerm_storage_account.storage
```

!!! danger "`terraform destroy` en production = catastrophe si mal utilisé"
    Toujours vérifier `terraform plan -destroy` avant. En prod, utilise `lifecycle { prevent_destroy = true }` sur les ressources critiques.

---

## Récap — Structure d'un projet Terraform

```
mon-projet/
├── main.tf           # Ressources principales
├── variables.tf      # Déclarations des variables
├── outputs.tf        # Valeurs affichées après apply
├── providers.tf      # Configuration des providers
├── terraform.tfvars  # Valeurs des variables (⚠️ ne pas commiter si secrets)
└── .gitignore        # Doit contenir : .terraform/, *.tfstate, *.tfstate.backup
```

!!! success "Checkpoint débutant ✅"
    Tu comprends : Provider, Resource, State, Variables, Outputs.
    Tu sais : init/plan/apply/destroy, créer un RG et un Storage Account.
