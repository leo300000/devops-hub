# 🟡 Terraform — Intermédiaire

!!! info "Documentation officielle"
    - [Modules Terraform](https://developer.hashicorp.com/terraform/language/modules)
    - [AzureRM — Virtual Network](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/virtual_network)
    - [AzureRM — AKS](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/kubernetes_cluster)
    - [Terraform Backend Azure](https://developer.hashicorp.com/terraform/language/settings/backends/azurerm)

---

## Remote State — Partager l'état en équipe

Par défaut, le state est **local** (ton PC). En équipe, il doit être dans Azure Blob Storage.

```bash
# Étape 1 — Créer le storage pour le state (une seule fois, à la main ou avec un script)
az group create --name rg-terraform-state --location francecentral
az storage account create \
  --name satfstate$(openssl rand -hex 4) \
  --resource-group rg-terraform-state \
  --sku Standard_LRS
az storage container create \
  --name tfstate \
  --account-name <nom-du-storage>
```

```hcl
# backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "satfstate1a2b3c4d"
    container_name       = "tfstate"
    key                  = "dev/terraform.tfstate"
    # En CI/CD : utiliser des variables d'env ARM_ACCESS_KEY ou ARM_CLIENT_SECRET
  }
}
```

```bash
terraform init
# Terraform migre automatiquement le state local vers Azure si besoin
```

!!! info "Locking automatique"
    Azure Blob Storage verrouille le state pendant un `apply`. Deux personnes ne peuvent pas modifier l'infra en même temps.

---

## Modules — Factoriser son code

!!! quote "Analogie"
    Un module = une **fonction réutilisable**. Tu l'écris une fois, tu l'appelles partout avec des paramètres différents.

### Structure d'un module

```
modules/
└── networking/
    ├── main.tf       # Les ressources
    ├── variables.tf  # Les inputs du module
    └── outputs.tf    # Les outputs du module
```

```hcl
# modules/networking/variables.tf
variable "resource_group_name" { type = string }
variable "location"            { type = string }
variable "vnet_address_space"  { type = list(string) }
variable "subnets" {
  type = map(object({
    address_prefix = string
  }))
}
```

```hcl
# modules/networking/main.tf
resource "azurerm_virtual_network" "vnet" {
  name                = "vnet-${var.resource_group_name}"
  resource_group_name = var.resource_group_name
  location            = var.location
  address_space       = var.vnet_address_space
}

resource "azurerm_subnet" "subnets" {
  for_each = var.subnets

  name                 = each.key
  resource_group_name  = var.resource_group_name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = [each.value.address_prefix]
}
```

```hcl
# modules/networking/outputs.tf
output "vnet_id"    { value = azurerm_virtual_network.vnet.id }
output "subnet_ids" { value = { for k, v in azurerm_subnet.subnets : k => v.id } }
```

### Appeler le module

```hcl
# main.tf — environnement dev
module "networking" {
  source = "./modules/networking"

  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  vnet_address_space  = ["10.0.0.0/16"]
  subnets = {
    "subnet-web"    = { address_prefix = "10.0.1.0/24" }
    "subnet-api"    = { address_prefix = "10.0.2.0/24" }
    "subnet-db"     = { address_prefix = "10.0.3.0/24" }
  }
}

# Utiliser les outputs du module
output "vnet_id" {
  value = module.networking.vnet_id
}
```

---

## Workspaces — Multi-environnements

```bash
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

terraform workspace list
# * dev
#   staging
#   prod

terraform workspace select prod
```

```hcl
# Utiliser le workspace dans le code
locals {
  env_config = {
    dev     = { vm_size = "Standard_B2s",  node_count = 1 }
    staging = { vm_size = "Standard_D4s_v3", node_count = 2 }
    prod    = { vm_size = "Standard_D8s_v3", node_count = 5 }
  }
  config = local.env_config[terraform.workspace]
}

resource "azurerm_resource_group" "rg" {
  name     = "rg-monapp-${terraform.workspace}"
  location = "France Central"
}
```

---

## `for_each` — Créer plusieurs ressources

```hcl
# Créer plusieurs subnets depuis une map
variable "subnets" {
  default = {
    "subnet-web" = "10.0.1.0/24"
    "subnet-api" = "10.0.2.0/24"
    "subnet-db"  = "10.0.3.0/24"
  }
}

resource "azurerm_subnet" "subnets" {
  for_each = var.subnets

  name                 = each.key    # "subnet-web", "subnet-api"...
  address_prefixes     = [each.value] # "10.0.1.0/24"...
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
}

# Accéder à un subnet spécifique
output "web_subnet_id" {
  value = azurerm_subnet.subnets["subnet-web"].id
}
```

!!! tip "`for_each` > `count`"
    Avec `count`, supprimer le 2e élément de 3 force la recréation du 3e.
    Avec `for_each`, chaque ressource est identifiée par sa clé — pas d'effet de bord.

---

## Data Sources — Lire des ressources existantes

```hcl
# Récupérer un Key Vault existant (créé hors Terraform)
data "azurerm_key_vault" "kv" {
  name                = "kv-mon-projet"
  resource_group_name = "rg-shared"
}

# Lire un secret depuis le Key Vault
data "azurerm_key_vault_secret" "db_password" {
  name         = "database-password"
  key_vault_id = data.azurerm_key_vault.kv.id
}

# Récupérer l'ID de la subscription courante
data "azurerm_client_config" "current" {}

# Utiliser dans une ressource
resource "azurerm_role_assignment" "example" {
  scope                = "/subscriptions/${data.azurerm_client_config.current.subscription_id}"
  role_definition_name = "Reader"
  principal_id         = data.azurerm_client_config.current.object_id
}
```

---

## `locals` — Variables calculées

```hcl
locals {
  # Convention de nommage standardisée
  prefix = "${var.project}-${var.environment}"

  # Tags communs à toutes les ressources
  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "Terraform"
    Owner       = "leo300000"
    CostCenter  = "IT-DevOps"
  }

  # Calculer un nom unique pour le storage account (doit être < 24 chars, sans tirets)
  storage_name = lower(replace("st${var.project}${var.environment}", "-", ""))
}

resource "azurerm_resource_group" "rg" {
  name = "rg-${local.prefix}"
  tags = local.common_tags
}

resource "azurerm_storage_account" "storage" {
  name = local.storage_name
  tags = local.common_tags
}
```

---

## Commandes utiles

```bash
terraform fmt              # Auto-formate le code HCL
terraform validate         # Vérifie la syntaxe (sans connexion cloud)
terraform state list       # Liste toutes les ressources dans le state
terraform state show azurerm_resource_group.rg  # Détails d'une ressource

# Importer une ressource créée manuellement dans Azure
terraform import \
  azurerm_resource_group.rg \
  /subscriptions/<sub-id>/resourceGroups/rg-existant

# Supprimer une ressource du state sans la détruire dans Azure
terraform state rm azurerm_storage_account.storage

# Voir le plan de destruction
terraform plan -destroy

# Appliquer sans confirmation interactive (CI/CD)
terraform apply -auto-approve
```

!!! success "Checkpoint intermédiaire ✅"
    Tu maîtrises : remote state, modules, workspaces, for_each, data sources, locals.
