# 🟢 Azure — Débutant

!!! info "Documentation officielle"
    - [Azure Docs](https://learn.microsoft.com/fr-fr/azure/)
    - [Azure CLI Reference](https://learn.microsoft.com/fr-fr/cli/azure/reference-index)
    - [Azure Pricing Calculator](https://azure.microsoft.com/fr-fr/pricing/calculator/)
    - [Azure Free Account](https://azure.microsoft.com/fr-fr/free/) — 200€ de crédits
    - [AZ-900 Certification](https://learn.microsoft.com/fr-fr/certifications/azure-fundamentals/)

---

## L'organisation hiérarchique Azure

```
Tenant (ton organisation / Azure AD)
└── Management Group (regroupement de subscriptions)
    └── Subscription (contrat de facturation = une carte bleue)
        └── Resource Group (dossier logique)
            ├── Virtual Machine
            ├── Storage Account
            ├── Virtual Network
            └── ...
```

!!! tip "La règle du Resource Group"
    Mets ensemble tout ce qui **appartient au même projet ou environnement**.
    Quand tu supprimes un resource group, tu supprimes tout ce qu'il contient.

---

## Azure vs AWS — Tableau de correspondance

| Catégorie | Azure | AWS | GCP |
|-----------|-------|-----|-----|
| **VM** | Virtual Machines | EC2 | Compute Engine |
| **Kubernetes** | AKS | EKS | GKE |
| **Serverless** | Azure Functions | Lambda | Cloud Functions |
| **BDD SQL** | Azure SQL / PostgreSQL | RDS | Cloud SQL |
| **BDD NoSQL** | Cosmos DB | DynamoDB | Firestore |
| **Stockage objet** | Blob Storage | S3 | GCS |
| **Réseau privé** | VNet | VPC | VPC |
| **Secrets** | Key Vault | Secrets Manager | Secret Manager |
| **Container Registry** | ACR | ECR | Artifact Registry |
| **CI/CD** | Azure DevOps / GitHub Actions | CodePipeline | Cloud Build |
| **Monitoring** | Azure Monitor | CloudWatch | Cloud Monitoring |
| **IAM** | Azure AD + RBAC | IAM | IAM |
| **DNS** | Azure DNS | Route 53 | Cloud DNS |

---

## Installer Azure CLI

```bash
# Windows (winget)
winget install Microsoft.AzureCLI

# Mac
brew install azure-cli

# Ubuntu
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Connexion
az login
# Ouvre le navigateur pour t'authentifier

# Voir tes subscriptions
az account list --output table

# Sélectionner une subscription
az account set --subscription "Mon-Abonnement"
az account show  # Vérifier la subscription active
```

---

## Commandes Azure CLI essentielles

```bash
# === Resource Groups ===
az group create \
  --name "rg-mon-projet-dev" \
  --location "francecentral"

az group list --output table
az group show --name "rg-mon-projet-dev"
az group delete --name "rg-mon-projet-dev" --yes --no-wait

# === Régions disponibles ===
az account list-locations --output table
# francecentral, francesouth, westeurope, northeurope...

# === Tags ===
az group update \
  --name "rg-mon-projet-dev" \
  --tags Environment=dev Project=monapp ManagedBy=Terraform
```

---

## Créer une VM

```bash
# Créer une VM Ubuntu
az vm create \
  --resource-group rg-mon-projet-dev \
  --name vm-web-01 \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-sku Standard

# Se connecter en SSH
IP=$(az vm show \
  --resource-group rg-mon-projet-dev \
  --name vm-web-01 \
  --show-details \
  --query publicIps -o tsv)
ssh azureuser@$IP

# Arrêter (facturé si juste stoppé)
az vm stop --resource-group rg-mon-projet-dev --name vm-web-01

# Désallouer (plus facturé pour le compute)
az vm deallocate --resource-group rg-mon-projet-dev --name vm-web-01

# Redémarrer
az vm start --resource-group rg-mon-projet-dev --name vm-web-01

# Supprimer
az vm delete --resource-group rg-mon-projet-dev --name vm-web-01 --yes
```

### Les tailles de VM

| Série | Usage | Exemple | vCPU | RAM |
|-------|-------|---------|------|-----|
| **B** | Dev/test, burst | Standard_B2s | 2 | 4 Go |
| **D** | Usage général prod | Standard_D4s_v3 | 4 | 16 Go |
| **E** | Mémoire intensive | Standard_E8s_v3 | 8 | 64 Go |
| **F** | CPU intensive | Standard_F4s_v2 | 4 | 8 Go |
| **L** | Stockage NVMe | Standard_L8s_v3 | 8 | 64 Go |
| **N** | GPU (ML/rendu) | Standard_NC6s_v3 | 6 | 112 Go |

---

## Blob Storage — Stocker des fichiers

```bash
# Créer un storage account
az storage account create \
  --name "stmonprojetdev001" \
  --resource-group rg-mon-projet-dev \
  --location francecentral \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot

# Créer un container (dossier)
az storage container create \
  --name "uploads" \
  --account-name "stmonprojetdev001" \
  --public-access off

# Uploader un fichier
az storage blob upload \
  --account-name "stmonprojetdev001" \
  --container-name "uploads" \
  --file ./photo.jpg \
  --name "photos/photo.jpg"

# Lister les fichiers
az storage blob list \
  --account-name "stmonprojetdev001" \
  --container-name "uploads" \
  --output table

# Télécharger
az storage blob download \
  --account-name "stmonprojetdev001" \
  --container-name "uploads" \
  --name "photos/photo.jpg" \
  --file ./photo-downloaded.jpg
```

### Les types de redondance

| SKU | Réplication | Usage | Prix |
|-----|------------|-------|------|
| **LRS** | 1 région, 3 copies | Dev/test | 💰 |
| **ZRS** | 3 zones de dispo | Production | 💰💰 |
| **GRS** | 2 régions | Business critical | 💰💰💰 |
| **GZRS** | ZRS + GRS | Maximum | 💰💰💰💰 |

---

## Azure Functions — Serverless

```bash
# Installer Azure Functions Core Tools
npm install -g azure-functions-core-tools@4

# Créer un projet
func init monprojet --python
cd monprojet

# Créer une fonction HTTP
func new --name HttpTrigger --template "HTTP trigger"
```

```python
# function_app.py
import azure.functions as func
import json

app = func.FunctionApp()

@app.function_name(name="api")
@app.route(route="hello/{name}", auth_level=func.AuthLevel.ANONYMOUS)
def http_trigger(req: func.HttpRequest) -> func.HttpResponse:
    name = req.route_params.get('name', 'World')
    return func.HttpResponse(
        json.dumps({"message": f"Hello, {name}!"}),
        mimetype="application/json"
    )
```

```bash
# Tester en local
func start
# → http://localhost:7071/api/hello/Leo

# Déployer sur Azure
az functionapp create \
  --resource-group rg-mon-projet-dev \
  --consumption-plan-location francecentral \
  --runtime python \
  --runtime-version 3.11 \
  --functions-version 4 \
  --name "func-monprojet-dev" \
  --storage-account "stmonprojetdev001"

func azure functionapp publish func-monprojet-dev
```

---

## Portail Azure — L'interface web

Pour débuter, le **portail Azure** (portal.azure.com) est très utile :

1. Barre de recherche en haut → tape le service que tu cherches
2. **Groupes de ressources** → vois tout ce que tu as créé
3. **Surveillance des coûts** → Cost Management → vois combien tu dépenses
4. **Cloud Shell** → terminal Azure directement dans le navigateur (pas besoin d'installer Azure CLI)

!!! success "Checkpoint débutant ✅"
    Tu sais naviguer dans Azure, créer des VMs, du Blob Storage et des Azure Functions.
