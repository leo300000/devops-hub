# 🟢 Azure — Débutant

## L'organisation des ressources

```
Tenant (ton entreprise)
└── Subscription (contrat de facturation)
    └── Resource Group (dossier logique)
        ├── Virtual Machine
        ├── Storage Account
        └── Virtual Network
```

!!! tip "La règle du Resource Group"
    Un resource group = un projet ou un environnement.
    Tout ce qui vit ensemble, meurt ensemble → mets dans le même RG.

---

## Les commandes Azure CLI essentielles

```bash
# Connexion
az login

# Voir ses subscriptions
az account list --output table
az account set --subscription "Mon-Abonnement"

# Resource Groups
az group create --name "rg-mon-projet" --location "francecentral"
az group list --output table
az group delete --name "rg-mon-projet"
```

---

## Créer une VM

```bash
az vm create \
  --resource-group rg-mon-projet \
  --name vm-web \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --output json

# Se connecter en SSH
az vm show -g rg-mon-projet -n vm-web --query publicIps -o tsv
ssh azureuser@<IP>

# Démarrer/Arrêter
az vm start  -g rg-mon-projet -n vm-web
az vm stop   -g rg-mon-projet -n vm-web
az vm deallocate -g rg-mon-projet -n vm-web  # Arrête ET libère (ne facture plus le compute)
```

---

## Les tailles de VM

| Série | Usage | Exemple |
|-------|-------|---------|
| **B** | Usage général, dev/test | Standard_B2s |
| **D** | Équilibrée production | Standard_D4s_v3 |
| **E** | Mémoire intensive | Standard_E8s_v3 |
| **F** | CPU intensive | Standard_F4s_v2 |
| **N** | GPU | Standard_NC6 |

---

## Blob Storage — Stocker des fichiers

```bash
# Créer un compte de stockage
az storage account create \
  --name "moncomptedestock123" \
  --resource-group rg-mon-projet \
  --location francecentral \
  --sku Standard_LRS

# Créer un container (comme un dossier)
az storage container create \
  --name "mon-container" \
  --account-name "moncomptedestock123"

# Uploader un fichier
az storage blob upload \
  --account-name "moncomptedestock123" \
  --container-name "mon-container" \
  --file ./monFichier.txt \
  --name "monFichier.txt"
```

---

## Azure Functions — Serverless

```python
# function_app.py
import azure.functions as func
import logging

app = func.FunctionApp()

@app.function_name(name="HttpTrigger")
@app.route(route="hello")
def http_trigger(req: func.HttpRequest) -> func.HttpResponse:
    name = req.params.get('name', 'World')
    return func.HttpResponse(f"Hello, {name}!")
```

```bash
# Déployer
func azure functionapp publish mon-function-app
```

!!! success "Checkpoint débutant ✅"
    Tu sais naviguer dans Azure, créer des VMs, du stockage et des Functions.
