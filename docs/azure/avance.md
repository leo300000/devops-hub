# 🔴 Azure — Avancé

## Landing Zone — Architecture entreprise

Une Landing Zone = socle Azure prêt à l'emploi pour accueillir des workloads.

```
Management Group (racine)
├── Platform
│   ├── Connectivity (Hub VNet, VPN, ExpressRoute)
│   ├── Identity (Active Directory)
│   └── Management (Log Analytics, Automation)
└── Landing Zones
    ├── Corp (accès on-prem)
    │   ├── Subscription Prod
    │   └── Subscription Dev
    └── Online (accès internet direct)
```

---

## Azure Policy — Gouvernance as Code

```json
// Policy : obliger le tag "Environment" sur toutes les ressources
{
  "mode": "Indexed",
  "policyRule": {
    "if": {
      "field": "tags['Environment']",
      "exists": "false"
    },
    "then": {
      "effect": "deny"
    }
  }
}
```

```bash
# Créer et assigner la policy
az policy definition create \
  --name "require-environment-tag" \
  --rules policy.json \
  --mode Indexed

az policy assignment create \
  --name "enforce-env-tag" \
  --policy "require-environment-tag" \
  --scope /subscriptions/<sub-id>
```

---

## Cost Management — Maîtriser les coûts

```bash
# Voir les coûts du mois en cours
az consumption usage list \
  --start-date 2024-01-01 \
  --end-date 2024-01-31 \
  --output table

# Créer un budget avec alerte
az consumption budget create \
  --budget-name "Budget-Prod-Mensuel" \
  --amount 1000 \
  --time-grain Monthly \
  --start-date 2024-01-01 \
  --end-date 2025-12-31 \
  --notifications "[{\"enabled\":true,\"operator\":\"GreaterThan\",\"threshold\":80,\"contactEmails\":[\"leo@example.com\"]}]"
```

**Bonnes pratiques pour réduire les coûts :**
- Utiliser les **Reserved Instances** (-72% vs pay-as-you-go)
- **Auto-shutdown** des VMs de dev la nuit
- **Spot instances** pour les workloads tolérantes aux interruptions
- **Rightsizing** : détecter les VMs surdimensionnées

---

## Private Endpoints — Sécuriser les accès

```bash
# Accéder à un Storage Account sans passer par internet
az network private-endpoint create \
  --name pe-storage \
  --resource-group rg-prod \
  --vnet-name vnet-prod \
  --subnet subnet-private \
  --private-connection-resource-id /subscriptions/.../storageAccounts/moncompte \
  --group-id blob \
  --connection-name conn-storage
```

---

## Architecture de référence — App 3-tiers

```
Internet
    ↓
Application Gateway (WAF)
    ↓
[subnet-web]  VM Scale Set (nginx)
    ↓
[subnet-app]  AKS (API)
    ↓
[subnet-db]   Azure Database for PostgreSQL (Private Endpoint)
    ↓
[Key Vault]   Secrets (via Managed Identity, sans credentials)
```

**Principes appliqués :**
- Zero Trust : chaque couche vérifie
- Managed Identity : plus de secrets à gérer
- Private Endpoints : tout reste dans le VNet
- WAF : protection contre les attaques web
