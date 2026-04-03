# 🔴 Terraform — Avancé

## Architecture multi-environnements

Structure recommandée pour un vrai projet :

```
infrastructure/
├── modules/              # Modules réutilisables
│   ├── networking/
│   ├── compute/
│   └── database/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
└── global/               # Ressources partagées (DNS, state storage)
```

---

## Terragrunt — DRY à l'extrême

Terragrunt évite de répéter la config backend dans chaque environnement :

```hcl
# terragrunt.hcl (racine)
remote_state {
  backend = "azurerm"
  config = {
    resource_group_name  = "rg-terraform"
    storage_account_name = "satfstate"
    container_name       = "tfstate"
    key                  = "${path_relative_to_include()}/terraform.tfstate"
  }
}
```

---

## Sécurité — Ne jamais stocker de secrets dans le code

```hcl
# ❌ Mauvais
resource "azurerm_key_vault_secret" "db_password" {
  value = "MonSuperMotDePasse123!"  # JAMAIS ça !
}

# ✅ Bon — Depuis une variable d'environnement
variable "db_password" {
  type      = string
  sensitive = true  # Ne s'affiche pas dans les logs
}

# ✅ Meilleur — Depuis Azure Key Vault
data "azurerm_key_vault_secret" "db_password" {
  name         = "db-password"
  key_vault_id = data.azurerm_key_vault.kv.id
}
```

---

## CI/CD avec GitHub Actions

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  push:
    branches: [main]
  pull_request:

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.7.0

      - name: Terraform Init
        run: terraform init
        env:
          ARM_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
          ARM_CLIENT_SECRET: ${{ secrets.AZURE_CLIENT_SECRET }}
          ARM_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
          ARM_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}

      - name: Terraform Plan
        run: terraform plan -out=tfplan
        if: github.event_name == 'pull_request'

      - name: Terraform Apply
        run: terraform apply -auto-approve tfplan
        if: github.ref == 'refs/heads/main'
```

---

## Policy as Code avec Sentinel / OPA

```hcl
# Exemple OPA — Forcer les tags sur toutes les ressources
package terraform.analysis

deny[msg] {
  resource := input.resource_changes[_]
  resource.change.actions[_] == "create"
  not resource.change.after.tags.Environment
  msg := sprintf("La ressource %v doit avoir un tag 'Environment'", [resource.address])
}
```

---

## Bonnes pratiques avancées

| Pratique | Pourquoi |
|----------|----------|
| Verrouiller les versions des providers | Évite les breaking changes |
| Un state par environnement | Isolation, moins de risque |
| `terraform plan` en PR, `apply` sur merge | Revue de code de l'infra |
| Tagging systématique | Coûts, gouvernance |
| Modules versionnés dans un registry | Réutilisabilité, stabilité |

```hcl
# Toujours verrouiller les versions
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "= 3.85.0"  # Version exacte en prod
    }
  }
}
```
