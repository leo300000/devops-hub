# 🔴 Terraform — Avancé

!!! info "Documentation officielle"
    - [Terraform Best Practices](https://developer.hashicorp.com/terraform/language/style)
    - [AzureRM Provider — Toutes les ressources](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
    - [Terraform Registry — Modules Azure](https://registry.terraform.io/namespaces/Azure)
    - [Azure CAF Naming](https://registry.terraform.io/providers/aztfmod/azurecaf/latest/docs)

---

## 🏗️ Infrastructure complète sur Azure — La logique

Voici comment on pense une infrastructure de A à Z avec Terraform.

### La philosophie : des couches

```
Couche 1 — Foundation    : Resource Groups, VNet, Subnets, NSG
Couche 2 — Shared        : Key Vault, ACR, Log Analytics
Couche 3 — Compute       : AKS, VMs, App Service
Couche 4 — Data          : PostgreSQL, Redis, Storage
Couche 5 — App           : Configurations app, DNS
```

Chaque couche dépend de la précédente. On les déploie dans l'ordre.

### Structure du projet

```
infrastructure/
├── modules/
│   ├── networking/          # VNet, Subnets, NSG, Private DNS
│   ├── security/            # Key Vault, identités managées
│   ├── monitoring/          # Log Analytics, Azure Monitor
│   ├── container-registry/  # ACR
│   └── aks/                 # Cluster AKS
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
└── global/
    └── state-storage/       # Le storage account pour le state Terraform
```

---

## Couche 1 — Networking complet

```hcl
# modules/networking/main.tf

resource "azurerm_virtual_network" "vnet" {
  name                = "vnet-${var.prefix}"
  location            = var.location
  resource_group_name = var.resource_group_name
  address_space       = [var.vnet_cidr]

  tags = var.tags
}

resource "azurerm_subnet" "subnets" {
  for_each = var.subnets

  name                 = each.key
  resource_group_name  = var.resource_group_name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = [each.value.cidr]

  # Pour AKS, certains subnets ont besoin de délégations
  dynamic "delegation" {
    for_each = lookup(each.value, "delegation", null) != null ? [each.value.delegation] : []
    content {
      name = delegation.value.name
      service_delegation {
        name = delegation.value.service
      }
    }
  }
}

# NSG par subnet
resource "azurerm_network_security_group" "nsgs" {
  for_each = var.subnets

  name                = "nsg-${each.key}"
  location            = var.location
  resource_group_name = var.resource_group_name
  tags                = var.tags
}

# Association NSG → Subnet
resource "azurerm_subnet_network_security_group_association" "assoc" {
  for_each = var.subnets

  subnet_id                 = azurerm_subnet.subnets[each.key].id
  network_security_group_id = azurerm_network_security_group.nsgs[each.key].id
}

# Règles NSG pour le subnet web (autoriser HTTP/HTTPS depuis internet)
resource "azurerm_network_security_rule" "allow_http" {
  name                        = "Allow-HTTP-HTTPS"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_ranges     = ["80", "443"]
  source_address_prefix       = "Internet"
  destination_address_prefix  = "*"
  resource_group_name         = var.resource_group_name
  network_security_group_name = azurerm_network_security_group.nsgs["subnet-web"].name
}
```

```hcl
# Appel dans environments/dev/main.tf
module "networking" {
  source = "../../modules/networking"

  prefix              = local.prefix
  location            = var.location
  resource_group_name = azurerm_resource_group.rg.name
  vnet_cidr           = "10.0.0.0/16"
  tags                = local.common_tags

  subnets = {
    "subnet-web" = { cidr = "10.0.1.0/24" }
    "subnet-api" = { cidr = "10.0.2.0/24" }
    "subnet-aks" = { cidr = "10.0.3.0/23" }  # /23 = 512 IPs pour AKS
    "subnet-db"  = { cidr = "10.0.5.0/24" }
    "subnet-pe"  = {                           # Private Endpoints
      cidr = "10.0.6.0/24"
      private_endpoint_network_policies_enabled = false
    }
  }
}
```

---

## Couche 2 — Key Vault + Managed Identity

```hcl
# modules/security/main.tf

# Identité managée pour AKS (pas besoin de credentials)
resource "azurerm_user_assigned_identity" "aks" {
  name                = "mi-aks-${var.prefix}"
  location            = var.location
  resource_group_name = var.resource_group_name
  tags                = var.tags
}

# Key Vault pour stocker les secrets
resource "azurerm_key_vault" "kv" {
  name                = "kv-${var.prefix}"           # Max 24 chars, unique
  location            = var.location
  resource_group_name = var.resource_group_name
  tenant_id           = data.azurerm_client_config.current.tenant_id
  sku_name            = "standard"

  # Autoriser l'accès depuis le VNet uniquement
  network_acls {
    default_action             = "Deny"
    bypass                     = "AzureServices"
    virtual_network_subnet_ids = [var.subnet_pe_id]
  }

  # RBAC mode (plus sécurisé que les Access Policies)
  enable_rbac_authorization = true

  tags = var.tags
}

# Donner accès au Key Vault à l'identité AKS
resource "azurerm_role_assignment" "aks_kv_reader" {
  scope                = azurerm_key_vault.kv.id
  role_definition_name = "Key Vault Secrets User"
  principal_id         = azurerm_user_assigned_identity.aks.principal_id
}

# Ajouter des secrets
resource "azurerm_key_vault_secret" "db_password" {
  name         = "database-password"
  value        = var.db_password  # Variable sensitive en input
  key_vault_id = azurerm_key_vault.kv.id

  lifecycle {
    ignore_changes = [value]  # Ne pas écraser si modifié manuellement
  }
}
```

---

## Couche 3 — AKS Cluster

```hcl
# modules/aks/main.tf

resource "azurerm_kubernetes_cluster" "aks" {
  name                = "aks-${var.prefix}"
  location            = var.location
  resource_group_name = var.resource_group_name
  dns_prefix          = var.prefix
  kubernetes_version  = var.kubernetes_version  # "1.28.5"

  # Node pool par défaut (system)
  default_node_pool {
    name                = "system"
    node_count          = var.system_node_count
    vm_size             = "Standard_D4s_v3"
    vnet_subnet_id      = var.subnet_aks_id
    os_disk_size_gb     = 100
    type                = "VirtualMachineScaleSets"
    enable_auto_scaling = true
    min_count           = 1
    max_count           = 5

    node_labels = {
      "nodepool-type" = "system"
    }
  }

  # Identité managée (pas de credentials à gérer !)
  identity {
    type         = "UserAssigned"
    identity_ids = [var.managed_identity_id]
  }

  # Azure CNI pour que les pods aient des IPs du VNet
  network_profile {
    network_plugin    = "azure"
    network_policy    = "calico"
    load_balancer_sku = "standard"
    service_cidr      = "172.16.0.0/16"
    dns_service_ip    = "172.16.0.10"
  }

  # Intégration Azure AD pour le RBAC K8s
  azure_active_directory_role_based_access_control {
    managed            = true
    azure_rbac_enabled = true
  }

  # Monitoring intégré
  oms_agent {
    log_analytics_workspace_id = var.log_analytics_workspace_id
  }

  # Key Vault intégration pour les secrets K8s
  key_vault_secrets_provider {
    secret_rotation_enabled  = true
    secret_rotation_interval = "2m"
  }

  tags = var.tags
}

# Node pool dédié pour les workloads (user)
resource "azurerm_kubernetes_cluster_node_pool" "user" {
  name                  = "user"
  kubernetes_cluster_id = azurerm_kubernetes_cluster.aks.id
  vm_size               = "Standard_D8s_v3"
  vnet_subnet_id        = var.subnet_aks_id
  enable_auto_scaling   = true
  min_count             = 2
  max_count             = 20
  os_disk_size_gb       = 128

  node_labels = {
    "nodepool-type" = "user"
  }

  node_taints = ["workload=user:NoSchedule"]  # Réservé aux workloads user
}

# Rôle pour que AKS puisse pull depuis ACR
resource "azurerm_role_assignment" "aks_acr_pull" {
  scope                = var.acr_id
  role_definition_name = "AcrPull"
  principal_id         = azurerm_kubernetes_cluster.aks.kubelet_identity[0].object_id
}
```

---

## Couche 4 — Base de données PostgreSQL

```hcl
# modules/database/main.tf

resource "azurerm_postgresql_flexible_server" "postgres" {
  name                   = "psql-${var.prefix}"
  resource_group_name    = var.resource_group_name
  location               = var.location
  version                = "15"
  administrator_login    = "psqladmin"
  administrator_password = var.db_password
  storage_mb             = 32768
  sku_name               = var.sku  # "B_Standard_B2ms" dev / "GP_Standard_D4s_v3" prod

  # Déploiement dans le VNet (pas d'accès public)
  delegated_subnet_id = var.subnet_db_id
  private_dns_zone_id = azurerm_private_dns_zone.postgres.id

  high_availability {
    mode                      = var.environment == "prod" ? "ZoneRedundant" : "Disabled"
    standby_availability_zone = "2"
  }

  backup_retention_days        = var.environment == "prod" ? 35 : 7
  geo_redundant_backup_enabled = var.environment == "prod" ? true : false

  tags = var.tags
}

# Zone DNS privée pour résoudre le nom du serveur depuis le VNet
resource "azurerm_private_dns_zone" "postgres" {
  name                = "privatelink.postgres.database.azure.com"
  resource_group_name = var.resource_group_name
}

resource "azurerm_private_dns_zone_virtual_network_link" "postgres" {
  name                  = "link-postgres"
  resource_group_name   = var.resource_group_name
  private_dns_zone_name = azurerm_private_dns_zone.postgres.name
  virtual_network_id    = var.vnet_id
}
```

---

## Le fichier principal — Tout assembler

```hcl
# environments/prod/main.tf

terraform {
  required_version = ">= 1.5.0"
  required_providers {
    azurerm = { source = "hashicorp/azurerm", version = "= 3.85.0" }
  }
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "satfstate1a2b3c4d"
    container_name       = "tfstate"
    key                  = "prod/terraform.tfstate"
  }
}

locals {
  prefix = "${var.project}-${var.environment}"
  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

resource "azurerm_resource_group" "rg" {
  name     = "rg-${local.prefix}"
  location = var.location
  tags     = local.common_tags
}

module "networking" {
  source              = "../../modules/networking"
  prefix              = local.prefix
  location            = var.location
  resource_group_name = azurerm_resource_group.rg.name
  tags                = local.common_tags
  vnet_cidr           = "10.0.0.0/16"
  subnets = {
    "subnet-aks" = { cidr = "10.0.0.0/22" }
    "subnet-db"  = { cidr = "10.0.4.0/24" }
    "subnet-pe"  = { cidr = "10.0.5.0/24" }
  }
}

module "monitoring" {
  source              = "../../modules/monitoring"
  prefix              = local.prefix
  location            = var.location
  resource_group_name = azurerm_resource_group.rg.name
  tags                = local.common_tags
}

module "security" {
  source              = "../../modules/security"
  prefix              = local.prefix
  location            = var.location
  resource_group_name = azurerm_resource_group.rg.name
  subnet_pe_id        = module.networking.subnet_ids["subnet-pe"]
  db_password         = var.db_password  # sensitive var
  tags                = local.common_tags
}

module "acr" {
  source              = "../../modules/container-registry"
  prefix              = local.prefix
  location            = var.location
  resource_group_name = azurerm_resource_group.rg.name
  sku                 = "Premium"  # Pour private endpoint
  tags                = local.common_tags
}

module "aks" {
  source                     = "../../modules/aks"
  prefix                     = local.prefix
  location                   = var.location
  resource_group_name        = azurerm_resource_group.rg.name
  subnet_aks_id              = module.networking.subnet_ids["subnet-aks"]
  managed_identity_id        = module.security.aks_identity_id
  log_analytics_workspace_id = module.monitoring.workspace_id
  acr_id                     = module.acr.acr_id
  kubernetes_version         = "1.28.5"
  system_node_count          = 3
  tags                       = local.common_tags
  depends_on                 = [module.networking, module.security]
}

module "database" {
  source              = "../../modules/database"
  prefix              = local.prefix
  location            = var.location
  resource_group_name = azurerm_resource_group.rg.name
  subnet_db_id        = module.networking.subnet_ids["subnet-db"]
  vnet_id             = module.networking.vnet_id
  db_password         = var.db_password
  environment         = var.environment
  tags                = local.common_tags
}
```

---

## Ordre de déploiement et logique

```bash
# 1. D'abord, créer le storage pour le state (AVANT d'utiliser le backend)
cd global/state-storage
terraform init && terraform apply

# 2. Déployer la fondation
cd environments/prod
terraform init    # Télécharge providers + configure le backend Azure
terraform plan    # Vérifier : networking, monitoring, security
terraform apply   # Couches 1 et 2

# 3. Vérifier avant de continuer
az aks list --output table
az keyvault list --output table

# 4. Récupérer le kubeconfig AKS
az aks get-credentials --resource-group rg-monapp-prod --name aks-monapp-prod
kubectl get nodes
```

---

## CI/CD avec GitHub Actions + OIDC (sans secrets)

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  pull_request:
    paths: ["infrastructure/**"]
  push:
    branches: [main]
    paths: ["infrastructure/**"]

permissions:
  id-token: write    # Pour OIDC
  contents: read
  pull-requests: write

jobs:
  terraform:
    runs-on: ubuntu-latest
    environment: production

    steps:
    - uses: actions/checkout@v4

    - name: Azure Login (OIDC — sans client_secret !)
      uses: azure/login@v2
      with:
        client-id: ${{ secrets.AZURE_CLIENT_ID }}
        tenant-id: ${{ secrets.AZURE_TENANT_ID }}
        subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

    - uses: hashicorp/setup-terraform@v3
      with:
        terraform_version: "1.7.0"

    - name: Terraform Init
      run: terraform init
      working-directory: infrastructure/environments/prod

    - name: Terraform Plan
      id: plan
      run: terraform plan -out=tfplan -no-color
      working-directory: infrastructure/environments/prod

    - name: Commenter le plan sur la PR
      if: github.event_name == 'pull_request'
      uses: actions/github-script@v7
      with:
        script: |
          const output = `### Terraform Plan 📖
          \`\`\`
          ${{ steps.plan.outputs.stdout }}
          \`\`\``;
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: output
          })

    - name: Terraform Apply
      if: github.ref == 'refs/heads/main'
      run: terraform apply -auto-approve tfplan
      working-directory: infrastructure/environments/prod
```

---

## Bonnes pratiques avancées

| Pratique | Pourquoi |
|----------|----------|
| Version exacte du provider en prod (`= 3.85.0`) | Pas de breaking change surprise |
| Un state par environnement | Isolation, blast radius limité |
| `sensitive = true` sur les outputs sensibles | Pas de secrets dans les logs |
| `lifecycle { prevent_destroy = true }` sur les BDD | Évite les suppressions accidentelles |
| `depends_on` explicite si nécessaire | Ordre de création garanti |
| Tagging systématique | Coûts, gouvernance, audit |

```hcl
# Protéger une ressource critique de la destruction
resource "azurerm_postgresql_flexible_server" "postgres" {
  # ...
  lifecycle {
    prevent_destroy = true  # terraform destroy échouera sur cette ressource
  }
}
```
