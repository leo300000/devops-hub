# 🟡 Azure — Intermédiaire

## Networking — VNet, Subnets, NSG

```bash
# Créer un VNet
az network vnet create \
  --name vnet-prod \
  --resource-group rg-prod \
  --address-prefix 10.0.0.0/16

# Créer des subnets
az network vnet subnet create \
  --name subnet-web \
  --vnet-name vnet-prod \
  --resource-group rg-prod \
  --address-prefix 10.0.1.0/24

az network vnet subnet create \
  --name subnet-db \
  --vnet-name vnet-prod \
  --resource-group rg-prod \
  --address-prefix 10.0.2.0/24
```

---

## NSG — Firewall des VMs

```bash
# Créer un NSG
az network nsg create \
  --name nsg-web \
  --resource-group rg-prod

# Autoriser HTTP et HTTPS
az network nsg rule create \
  --nsg-name nsg-web \
  --resource-group rg-prod \
  --name AllowHTTP \
  --priority 100 \
  --protocol Tcp \
  --destination-port-ranges 80 443 \
  --access Allow

# Bloquer tout le reste (déjà par défaut)
```

---

## IAM — Qui a accès à quoi

```bash
# Voir les rôles disponibles
az role definition list --output table

# Assigner un rôle
az role assignment create \
  --assignee user@example.com \
  --role "Contributor" \
  --scope /subscriptions/<sub-id>/resourceGroups/rg-prod

# Service Principal pour les apps/scripts
az ad sp create-for-rbac \
  --name "sp-terraform" \
  --role "Contributor" \
  --scopes /subscriptions/<sub-id>
```

| Rôle | Peut faire |
|------|-----------|
| **Owner** | Tout + gérer les accès |
| **Contributor** | Créer/modifier/supprimer des ressources |
| **Reader** | Voir uniquement |
| **Custom** | Permissions sur mesure |

---

## Key Vault — Gérer les secrets

```bash
# Créer un Key Vault
az keyvault create \
  --name "kv-mon-projet-prod" \
  --resource-group rg-prod \
  --location francecentral

# Ajouter un secret
az keyvault secret set \
  --vault-name "kv-mon-projet-prod" \
  --name "DatabasePassword" \
  --value "MonSuperMotDePasse"

# Lire un secret
az keyvault secret show \
  --vault-name "kv-mon-projet-prod" \
  --name "DatabasePassword" \
  --query value -o tsv
```

---

## AKS — Kubernetes managé

```bash
# Créer un cluster AKS
az aks create \
  --resource-group rg-prod \
  --name aks-prod \
  --node-count 3 \
  --node-vm-size Standard_D4s_v3 \
  --enable-addons monitoring \
  --generate-ssh-keys

# Se connecter
az aks get-credentials \
  --resource-group rg-prod \
  --name aks-prod

kubectl get nodes  # Tu es connecté à ton cluster !
```

---

## Azure Monitor — Alertes

```bash
# Créer une alerte CPU > 80%
az monitor metrics alert create \
  --name "Alert-CPU-High" \
  --resource-group rg-prod \
  --scopes /subscriptions/<sub>/resourceGroups/rg-prod/providers/Microsoft.Compute/virtualMachines/vm-web \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action email leo@example.com
```

!!! success "Checkpoint intermédiaire ✅"
    Tu maîtrises : VNet/NSG, IAM, Key Vault, AKS et Azure Monitor.
