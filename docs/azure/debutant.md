# 🟢 Azure — Débutant

!!! info "Documentation officielle"
    - [Azure Docs](https://learn.microsoft.com/fr-fr/azure/)
    - [Azure CLI Reference](https://learn.microsoft.com/fr-fr/cli/azure/reference-index)
    - [Azure Pricing Calculator](https://azure.microsoft.com/fr-fr/pricing/calculator/)
    - [Azure Free Account](https://azure.microsoft.com/fr-fr/free/) — 200€ de crédits
    - [AZ-900 Certification](https://learn.microsoft.com/fr-fr/certifications/azure-fundamentals/)

---

## C'est quoi Azure ?

Azure est le **cloud public de Microsoft**. Au lieu d'acheter et maintenir des serveurs physiques, tu loues des ressources informatiques (serveurs, stockage, réseau, bases de données...) à la demande, et tu paies uniquement ce que tu utilises.

!!! quote "Analogie"
    Azure, c'est l'**électricité du réseau** appliquée à l'informatique.
    Tu n'as pas ta propre centrale électrique chez toi. Tu te branches sur le réseau et tu paies ce que tu consommes. Azure = la centrale. Ton app = ton appartement.

**Sans cloud :**
```
Achète des serveurs → Installe → Configure → Maintiens → Remplace tous les 5 ans
→ Paie même quand personne ne s'en sert → Sous-dimensionné en pic, surdimensionné le reste du temps
```

**Avec Azure :**
```
Crée une VM en 2 minutes → Arrête la nuit si inutile → Scale à la demande
→ Paie à l'usage → Microsoft gère le hardware
```

---

## L'organisation hiérarchique — Comment Azure est structuré

Avant de créer quoi que ce soit, comprends cette hiérarchie :

```
Azure Active Directory (Tenant)
│  ← Ton organisation / ton compte Microsoft
│
└── Management Group
    │  ← Regroupe plusieurs subscriptions (utile en entreprise)
    │
    └── Subscription
        │  ← Contrat de facturation = une carte bleue
        │  ← Tout ce que tu crées dans Azure appartient à une subscription
        │
        └── Resource Group
            │  ← Dossier logique qui regroupe des ressources liées
            │
            ├── Virtual Machine
            ├── Storage Account
            ├── Virtual Network
            └── ...
```

### Resource Group — C'est quoi ?

Un Resource Group est un **dossier logique** dans Azure. Il regroupe des ressources qui appartiennent au même projet ou au même environnement.

**Règles du Resource Group :**
- Toute ressource Azure DOIT appartenir à un Resource Group
- Quand tu supprimes un Resource Group, tu supprimes **tout ce qu'il contient**
- C'est l'unité de facturation et de gestion des droits d'accès

!!! quote "Analogie"
    Le Resource Group = un **classeur de bureau**.
    Tout ce qui concerne le projet "Monapp-Prod" est dans le même classeur. Tu jettes le classeur entier quand le projet est terminé.

```bash
# Bonne pratique de nommage
rg-monapp-dev        # rg = resource group, monapp = projet, dev = environnement
rg-monapp-staging
rg-monapp-prod
rg-shared-services   # Ressources partagées entre plusieurs projets
```

---

## Azure vs AWS vs GCP — Tableau de correspondance

| Catégorie | Azure | AWS | GCP |
|-----------|-------|-----|-----|
| **VM** | Virtual Machines | EC2 | Compute Engine |
| **Kubernetes** | AKS | EKS | GKE |
| **Serverless** | Azure Functions | Lambda | Cloud Functions |
| **BDD SQL** | Azure SQL / PostgreSQL Flexible | RDS | Cloud SQL |
| **BDD NoSQL** | Cosmos DB | DynamoDB | Firestore |
| **Stockage objet** | Blob Storage | S3 | GCS |
| **Réseau privé** | Virtual Network (VNet) | VPC | VPC |
| **Secrets** | Key Vault | Secrets Manager | Secret Manager |
| **Container Registry** | ACR | ECR | Artifact Registry |
| **CI/CD** | Azure DevOps / GitHub Actions | CodePipeline | Cloud Build |
| **Monitoring** | Azure Monitor / Log Analytics | CloudWatch | Cloud Monitoring |
| **DNS** | Azure DNS | Route 53 | Cloud DNS |
| **CDN** | Azure Front Door | CloudFront | Cloud CDN |

---

## Installer Azure CLI

Azure CLI (`az`) est l'outil en ligne de commande pour gérer Azure depuis ton terminal.

```bash
# Windows
winget install Microsoft.AzureCLI

# Mac
brew install azure-cli

# Ubuntu
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Connexion (ouvre le navigateur)
az login

# Voir toutes tes subscriptions
az account list --output table

# Sélectionner la bonne subscription
az account set --subscription "Mon-Abonnement"

# Vérifier qu'on est sur la bonne
az account show --query "{Name:name, ID:id}" -o table
```

---

## Virtual Machine — C'est quoi ?

Une VM (Virtual Machine) est un **serveur virtuel** dans le cloud. Elle tourne sur du hardware physique chez Microsoft, mais elle se comporte comme un serveur dédié. Tu peux y installer tout ce que tu veux : Ubuntu, Windows Server, Nginx, PostgreSQL...

```bash
# Créer un Resource Group d'abord
az group create \
  --name "rg-mon-projet-dev" \
  --location "francecentral"

# Créer une VM Ubuntu
az vm create \
  --resource-group rg-mon-projet-dev \
  --name vm-web-01 \
  --image Ubuntu2204 \         # Image du système d'exploitation
  --size Standard_B2s \        # Taille de la VM (2 vCPU, 4 Go RAM)
  --admin-username azureuser \ # Nom d'utilisateur SSH
  --generate-ssh-keys \        # Génère une paire de clés SSH automatiquement
  --public-ip-sku Standard     # Crée une IP publique Standard

# Récupérer l'IP publique
IP=$(az vm show \
  --resource-group rg-mon-projet-dev \
  --name vm-web-01 \
  --show-details \
  --query publicIps -o tsv)

# Se connecter en SSH
ssh azureuser@$IP
```

### Les tailles de VM — Comment choisir ?

| Série | Usage | Exemple | vCPU | RAM | Cas d'usage |
|-------|-------|---------|------|-----|-------------|
| **B** | Dev/test, usage faible | Standard_B2s | 2 | 4 Go | Dev, petits sites |
| **D** | Usage général prod | Standard_D4s_v3 | 4 | 16 Go | Web apps, API |
| **E** | Mémoire intensive | Standard_E8s_v3 | 8 | 64 Go | Bases de données, cache |
| **F** | CPU intensive | Standard_F4s_v2 | 4 | 8 Go | Calcul, batch |
| **L** | Stockage NVMe rapide | Standard_L8s_v3 | 8 | 64 Go | BDD haute perf |
| **N** | GPU | Standard_NC6s_v3 | 6 | 112 Go | ML, rendu 3D |

```bash
# Lister toutes les tailles disponibles dans une région
az vm list-sizes --location francecentral --output table

# Gérer le cycle de vie de la VM
az vm stop       --resource-group rg-mon-projet-dev --name vm-web-01  # Arrête (facturé !)
az vm deallocate --resource-group rg-mon-projet-dev --name vm-web-01  # Arrête + libère (plus facturé pour le compute)
az vm start      --resource-group rg-mon-projet-dev --name vm-web-01  # Redémarre
az vm delete     --resource-group rg-mon-projet-dev --name vm-web-01 --yes
```

---

## Blob Storage — C'est quoi ?

Azure Blob Storage est le service de **stockage d'objets** (fichiers) d'Azure. Il stocke n'importe quel type de fichier : images, vidéos, backups, logs, fichiers statiques d'un site web...

!!! quote "Analogie"
    Blob Storage = un **Google Drive** accessible par API, ultra-scalable, pas de limite de taille.

**Structure :**
```
Storage Account (le compte de stockage)
└── Container (comme un dossier)
    ├── fichier1.jpg
    ├── backup-2024-01-15.tar.gz
    └── sous-dossier/
        └── fichier2.pdf
```

```bash
# Créer un Storage Account
az storage account create \
  --name "stmonprojetdev001" \   # Unique globalement, 3-24 chars, minuscules + chiffres
  --resource-group rg-mon-projet-dev \
  --location francecentral \
  --sku Standard_LRS \           # Type de redondance (voir tableau ci-dessous)
  --kind StorageV2 \             # Version du storage account (toujours StorageV2)
  --access-tier Hot              # Hot = accès fréquent / Cool = accès rare / Archive

# Créer un container (dossier)
az storage container create \
  --name "uploads" \
  --account-name "stmonprojetdev001" \
  --public-access off            # off / blob (fichiers publics) / container (tout public)

# Uploader un fichier
az storage blob upload \
  --account-name "stmonprojetdev001" \
  --container-name "uploads" \
  --file ./photo.jpg \
  --name "photos/photo.jpg"      # Chemin dans le container

# Lister les fichiers
az storage blob list \
  --account-name "stmonprojetdev001" \
  --container-name "uploads" \
  --output table
```

### Les types de redondance — Comment choisir ?

| SKU | Réplication | Protection | Prix relatif |
|-----|------------|-----------|-------------|
| **LRS** | 3 copies dans 1 datacenter | Panne serveur | 💰 Le moins cher |
| **ZRS** | 3 copies dans 3 zones de dispo | Panne d'une zone | 💰💰 |
| **GRS** | LRS + copie dans une 2e région | Catastrophe régionale | 💰💰💰 |
| **GZRS** | ZRS + copie dans une 2e région | Maximum de protection | 💰💰💰💰 |

**Règle pratique :** LRS pour dev/test, ZRS ou GRS pour la production.

---

## Azure Functions — C'est quoi ?

Azure Functions est un service **serverless**. Tu écris juste une fonction Python/JS/C#, et Azure s'occupe de tout le reste (serveur, scaling, disponibilité). Tu paies uniquement quand la fonction s'exécute.

!!! quote "Analogie"
    Serverless = embaucher un **freelance à la tâche**.
    Il travaille uniquement quand tu en as besoin, tu paies seulement ce qu'il fait. Pas de serveur à maintenir "pour rien" quand il n'y a pas de travail.

```python
# function_app.py — une Function HTTP simple
import azure.functions as func

app = func.FunctionApp()

@app.function_name(name="api")
@app.route(route="hello/{name}", auth_level=func.AuthLevel.ANONYMOUS)
def http_trigger(req: func.HttpRequest) -> func.HttpResponse:
    # Cette fonction est appelée à chaque requête HTTP GET/POST sur /api/hello/{name}
    name = req.route_params.get('name', 'World')
    return func.HttpResponse(f"Bonjour, {name} !")
```

```bash
# Installer les outils
npm install -g azure-functions-core-tools@4

# Créer un projet Function
func init monprojet --python && cd monprojet

# Tester en local
func start
# → http://localhost:7071/api/hello/Leo → "Bonjour, Leo !"

# Déployer sur Azure
az functionapp create \
  --resource-group rg-mon-projet-dev \
  --consumption-plan-location francecentral \
  --runtime python --runtime-version 3.11 \
  --functions-version 4 \
  --name "func-monprojet-dev" \
  --storage-account "stmonprojetdev001"

func azure functionapp publish func-monprojet-dev
```

!!! success "Checkpoint débutant ✅"
    Tu comprends : Resource Group, VM, Blob Storage, Azure Functions, la hiérarchie Azure.
    Tu sais : utiliser Azure CLI, créer et gérer les ressources de base.
