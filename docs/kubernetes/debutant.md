# 🟢 Kubernetes — Débutant

!!! info "Documentation officielle"
    - [Kubernetes Docs](https://kubernetes.io/docs/home/)
    - [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
    - [Kubernetes Playground](https://killercoda.com/playgrounds/scenario/kubernetes)

---

## C'est quoi Kubernetes ?

Kubernetes (K8s) est un **orchestrateur de containers**. Il gère automatiquement le démarrage, l'arrêt, la mise à l'échelle et la récupération de tes containers Docker.

!!! quote "Analogie"
    Docker = un musicien qui joue seul.
    Kubernetes = le chef d'orchestre qui coordonne 100 musiciens, s'assure que chacun joue au bon moment, et remplace instantanément un musicien qui s'évanouit.

**Sans Kubernetes :**
```
Container plante → app down → quelqu'un le redémarre manuellement à 3h du matin 😴
Trop de trafic  → app lente → quelqu'un ajoute des serveurs à la main
Mise à jour     → downtime  → les utilisateurs voient une erreur
```

**Avec Kubernetes :**
```
Container plante → K8s le redémarre en 2 secondes, personne ne s'en rend compte ✅
Trop de trafic  → K8s crée automatiquement plus d'instances ✅
Mise à jour     → K8s remplace les containers un par un, zéro downtime ✅
```

---

## Les objets Kubernetes — Vue d'ensemble

Avant de rentrer dans le code, comprends ces 6 objets. Tout le reste en découle.

### Pod — La plus petite unité

**C'est quoi ?** Un Pod est un groupe d'un ou plusieurs containers qui partagent le même réseau et stockage. C'est l'unité de base de Kubernetes — tout tourne dans des pods.

!!! quote "Analogie"
    Un Pod = un appartement. Les containers sont les colocataires qui partagent l'adresse (IP) et les pièces communes (volumes).

```
Pod
├── Container principal (ton app)
└── Container sidecar (logs, proxy... optionnel)
```

!!! warning "On ne crée jamais un Pod directement"
    En pratique, on ne crée pas des Pods à la main. On crée des **Deployments** qui gèrent les Pods automatiquement.

---

### Deployment — Le gestionnaire de Pods

**C'est quoi ?** Un Deployment est un objet qui dit à Kubernetes : *"Je veux X copies de ce container, maintiens-les en vie, et gère les mises à jour proprement."*

Il répond à 3 questions :
- **Quoi lancer ?** → quelle image Docker
- **Combien ?** → nombre de replicas
- **Comment mettre à jour ?** → stratégie de rolling update

!!! quote "Analogie"
    Un Deployment = un **contrat de travail** passé avec Kubernetes.
    Tu lui dis "je veux 3 serveurs Nginx en permanence". Si l'un tombe, Kubernetes en recrée un immédiatement pour respecter le contrat.

```yaml
# deployment.yaml
apiVersion: apps/v1       # La version de l'API K8s pour cet objet
kind: Deployment          # Le type d'objet qu'on crée

metadata:
  name: mon-app           # Le nom du Deployment dans K8s
  namespace: default      # Le "dossier" d'isolation (voir Namespace plus bas)
  labels:
    app: mon-app          # Étiquettes pour identifier/filtrer cet objet

spec:                     # La description de ce qu'on veut
  replicas: 3             # Je veux 3 copies (pods) de mon app en permanence
  
  selector:               # Comment le Deployment sait quels Pods il gère
    matchLabels:
      app: mon-app        # Il gère tous les pods qui ont ce label

  template:               # Le modèle pour créer les Pods
    metadata:
      labels:
        app: mon-app      # Ce label DOIT correspondre au selector ci-dessus
    
    spec:                 # Description du Pod et de ses containers
      containers:
      - name: mon-app     # Nom du container dans le pod
        image: nginx:1.25 # L'image Docker à utiliser (TOUJOURS préciser la version !)
        ports:
        - containerPort: 80  # Le port sur lequel le container écoute

        # Ressources — TOUJOURS les définir, sinon K8s ne peut pas scheduler
        resources:
          requests:       # Minimum garanti — K8s choisit un node avec au moins ça
            memory: "64Mi"
            cpu: "100m"   # 100 millicores = 0.1 CPU (1000m = 1 CPU entier)
          limits:         # Maximum autorisé — K8s tue le container si dépassé
            memory: "256Mi"
            cpu: "500m"

        # Variables d'environnement injectées dans le container
        env:
        - name: APP_ENV
          value: "production"
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:    # Récupère la valeur depuis un ConfigMap
              name: app-config  # Nom du ConfigMap
              key: database_host # Clé dans le ConfigMap
```

```bash
# Appliquer le fichier — K8s crée ou met à jour le Deployment
kubectl apply -f deployment.yaml

# Vérifier que les 3 pods sont bien créés et Running
kubectl get pods
# NAME                       READY   STATUS    RESTARTS   AGE
# mon-app-5d9f7f5c4-abc12   1/1     Running   0          30s
# mon-app-5d9f7f5c4-def34   1/1     Running   0          30s
# mon-app-5d9f7f5c4-ghi56   1/1     Running   0          30s

# Voir le Deployment
kubectl get deployments
# NAME      READY   UP-TO-DATE   AVAILABLE   AGE
# mon-app   3/3     3            3           1m
```

---

### Service — L'adresse stable de tes Pods

**C'est quoi ?** Les Pods sont éphémères — ils peuvent être créés, supprimés, déplacés sur d'autres nodes. Leur IP change à chaque fois. Un **Service** fournit une adresse stable (IP + DNS) qui pointe toujours vers les bons pods, peu importe leurs IPs internes.

!!! quote "Analogie"
    Les Pods = des employés qui changent de bureau régulièrement.
    Le Service = le numéro de téléphone du service client qui redirige toujours vers un employé disponible, peu importe lequel.

```yaml
# service.yaml
apiVersion: v1
kind: Service

metadata:
  name: mon-app-service     # Le DNS interne sera : mon-app-service.default.svc.cluster.local

spec:
  selector:
    app: mon-app            # Envoie le trafic vers tous les pods avec ce label
                            # (correspond aux labels du Deployment)
  ports:
  - name: http
    port: 80                # Le port du Service (ce que les autres apps appellent)
    targetPort: 80          # Le port du container (où le trafic est redirigé)
  
  type: ClusterIP           # Voir tableau ci-dessous
```

### Les types de Service expliqués

| Type | C'est quoi | Usage |
|------|-----------|-------|
| `ClusterIP` | IP interne au cluster, invisible de l'extérieur | Communication entre microservices |
| `NodePort` | Expose un port sur chaque node (30000-32767) | Tests rapides |
| `LoadBalancer` | Azure crée un vrai load balancer avec IP publique | Production, accès internet |

```bash
kubectl apply -f service.yaml
kubectl get services
# NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
# mon-app-service   ClusterIP   10.0.0.100    <none>        80/TCP    1m
```

---

### Namespace — Le dossier d'isolation

**C'est quoi ?** Un Namespace est un espace de noms virtuel dans le cluster. Il permet d'isoler des ressources — comme des dossiers dans un système de fichiers. On s'en sert pour séparer les environnements ou les équipes sur un même cluster.

!!! quote "Analogie"
    Un Namespace = un **appartement dans un immeuble**.
    Chaque appartement (namespace) a ses propres meubles (pods, services...). Les voisins ne se dérangent pas mutuellement.

```bash
# Créer des namespaces
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace prod

# Déployer dans un namespace spécifique
kubectl apply -f deployment.yaml -n dev

# Voir les ressources d'un namespace
kubectl get pods -n dev
kubectl get all -n dev

# Changer le namespace par défaut (évite de taper -n à chaque fois)
kubectl config set-context --current --namespace=dev
```

---

### ConfigMap — La configuration sans secrets

**C'est quoi ?** Un ConfigMap stocke des données de configuration sous forme de clés/valeurs. Il permet de **séparer la configuration du code** — tu changes la config sans rebuilder l'image Docker.

!!! quote "Analogie"
    Un ConfigMap = un **fichier `.env`** mais géré par Kubernetes, injectable dans les containers.

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  database_host: "postgres-service"   # Clé: Valeur
  database_port: "5432"
  app_env: "production"
  max_connections: "100"
  
  # On peut même stocker un fichier entier
  config.json: |
    {
      "maxRetries": 3,
      "timeout": 30
    }
```

---

### Secret — La configuration sensible

**C'est quoi ?** Un Secret est comme un ConfigMap, mais pour les données **sensibles** (mots de passe, tokens, certificats). K8s les stocke encodés en base64 et les protège mieux que les ConfigMaps.

!!! warning "Base64 ≠ chiffrement"
    Base64 est un encodage, pas un chiffrement. N'importe qui avec accès au cluster peut décoder les secrets. En production, utilise Azure Key Vault avec le CSI Driver.

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:                         # stringData = K8s encode automatiquement en base64
  database_password: "MonMotDePasse"
  api_key: "sk-abc123xyz"
```

```yaml
# Utiliser Secret dans un Deployment
containers:
- name: mon-app
  envFrom:
  - configMapRef:
      name: app-config        # Injecte toutes les clés du ConfigMap comme variables d'env
  - secretRef:
      name: app-secrets       # Injecte toutes les clés du Secret comme variables d'env
```

---

## Les commandes essentielles

```bash
# Voir les ressources
kubectl get pods                        # Pods du namespace courant
kubectl get pods -o wide                # Avec IP et node
kubectl get all                         # Tout (pods, services, deployments...)
kubectl get pods --watch                # Surveiller en temps réel

# Inspecter
kubectl describe pod mon-pod            # Détails + events (utile pour débugger)
kubectl describe deployment mon-app     # Détails du deployment
kubectl logs mon-pod                    # Logs du container
kubectl logs mon-pod -f                 # Logs en temps réel (comme tail -f)
kubectl logs mon-pod --previous         # Logs avant le dernier crash

# Interagir
kubectl exec -it mon-pod -- bash        # Shell interactif dans le container
kubectl exec mon-pod -- env             # Voir les variables d'environnement
kubectl port-forward pod/mon-pod 8080:80  # Accès local temporaire sans Service

# Créer/modifier/supprimer
kubectl apply -f fichier.yaml           # Créer ou mettre à jour
kubectl delete -f fichier.yaml          # Supprimer via le fichier
kubectl delete pod mon-pod              # Supprimer directement (le Deployment le recrée !)
```

---

## Mettre à jour une app — Rolling Update

**C'est quoi ?** Un Rolling Update remplace les vieux pods par les nouveaux **un par un** (ou par groupes). À aucun moment tous les pods ne sont down — zéro downtime.

```
Avant :  [v1] [v1] [v1]
Step 1 : [v2] [v1] [v1]    ← un nouveau pod créé, un ancien supprimé
Step 2 : [v2] [v2] [v1]
Après :  [v2] [v2] [v2]    ← mise à jour terminée
```

```bash
# Changer l'image = déclenche un rolling update
kubectl set image deployment/mon-app mon-app=nginx:1.26

# Voir la progression
kubectl rollout status deployment/mon-app
# Waiting for deployment "mon-app" rollout to finish: 1 out of 3 new replicas updated...
# deployment "mon-app" successfully rolled out

# Voir l'historique des déploiements
kubectl rollout history deployment/mon-app

# Annuler si ça se passe mal (revient à la version précédente)
kubectl rollout undo deployment/mon-app
```

!!! success "Checkpoint débutant ✅"
    Tu comprends : Pod, Deployment, Service, Namespace, ConfigMap, Secret.
    Tu sais : déployer une app, l'exposer, la mettre à jour et annuler un déploiement.
