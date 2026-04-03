# 🟢 Kubernetes — Débutant

!!! info "Documentation officielle"
    - [Kubernetes Docs](https://kubernetes.io/docs/home/)
    - [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
    - [Kubernetes Playground](https://killercoda.com/playgrounds/scenario/kubernetes)
    - [AKS Quickstart](https://learn.microsoft.com/azure/aks/learn/quick-kubernetes-deploy-cli)

---

## Installer kubectl

```bash
# Windows (winget)
winget install Kubernetes.kubectl

# Mac
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -sL https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/

# Vérifier
kubectl version --client
```

---

## Les concepts de base

| Objet | Analogie | Rôle |
|-------|----------|------|
| **Node** | Un serveur | Machine physique ou VM dans le cluster |
| **Pod** | Un container (ou groupe) | Plus petite unité deployable |
| **Deployment** | Recette de pods | Gère les replicas et les updates |
| **Service** | Adresse permanente | Expose les pods (load balancer interne) |
| **Namespace** | Dossier d'isolation | Sépare les environnements/équipes |
| **ConfigMap** | Fichier de config | Variables de config non-sensibles |
| **Secret** | Coffre-fort | Mots de passe, tokens, certificats |

---

## Les commandes essentielles

```bash
# === VOIR les ressources ===
kubectl get pods                        # Pods dans le namespace courant
kubectl get pods -n kube-system         # Dans le namespace kube-system
kubectl get pods --all-namespaces       # Partout
kubectl get pods -o wide                # Avec plus d'infos (IP, node...)
kubectl get all                         # Tout voir d'un coup

kubectl get deployments
kubectl get services
kubectl get nodes

# === INSPECTER ===
kubectl describe pod mon-pod            # Détails complets + events
kubectl describe node nom-du-node
kubectl logs mon-pod                    # Logs en temps réel
kubectl logs mon-pod --previous         # Logs du container précédent (après crash)
kubectl logs mon-pod -f                 # Follow (comme tail -f)
kubectl logs mon-pod -c mon-container   # Si plusieurs containers dans le pod

# === INTERAGIR ===
kubectl exec -it mon-pod -- bash        # Shell interactif
kubectl exec mon-pod -- env             # Exécuter une commande
kubectl port-forward pod/mon-pod 8080:80  # Accéder en local sans service

# === APPLIQUER ===
kubectl apply -f deployment.yaml       # Créer ou mettre à jour
kubectl delete -f deployment.yaml      # Supprimer
kubectl delete pod mon-pod             # Supprimer un pod (sera recréé par le deployment)
```

---

## Ton premier Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mon-app
  namespace: default
  labels:
    app: mon-app
spec:
  replicas: 3                        # 3 instances en parallèle
  selector:
    matchLabels:
      app: mon-app                   # Connecte le deployment aux pods via ce label
  template:
    metadata:
      labels:
        app: mon-app
    spec:
      containers:
      - name: mon-app
        image: nginx:1.25
        ports:
        - containerPort: 80

        # Ressources — TOUJOURS les définir
        resources:
          requests:                  # Minimum garanti
            memory: "64Mi"
            cpu: "100m"             # 100m = 0.1 CPU
          limits:                   # Maximum autorisé
            memory: "256Mi"
            cpu: "500m"

        # Variables d'environnement
        env:
        - name: APP_ENV
          value: "production"
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database_host
```

```bash
kubectl apply -f deployment.yaml
kubectl get pods      # 3 pods en cours de démarrage
# mon-app-5d9f7f5c4-abc12   1/1   Running   0   30s
# mon-app-5d9f7f5c4-def34   1/1   Running   0   30s
# mon-app-5d9f7f5c4-ghi56   1/1   Running   0   30s
```

---

## Service — Exposer ton app

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mon-app-service
spec:
  selector:
    app: mon-app               # Sélectionne les pods avec ce label
  ports:
  - name: http
    port: 80                   # Port du service
    targetPort: 80             # Port du container
  type: ClusterIP              # Interne au cluster seulement
```

```yaml
# Pour exposer sur internet (Azure crée un Load Balancer)
type: LoadBalancer
```

```bash
kubectl apply -f service.yaml
kubectl get services
# mon-app-service   ClusterIP   10.0.0.100   <none>   80/TCP   1m

# Tester sans quitter le cluster
kubectl run test --image=curlimages/curl --rm -it -- curl http://mon-app-service
```

### Les types de Service

| Type | Accès depuis | Usage |
|------|-------------|-------|
| `ClusterIP` | Intérieur cluster seulement | Communication entre microservices |
| `NodePort` | IP du node + port (30000-32767) | Dev/test |
| `LoadBalancer` | IP publique (Azure crée un LB) | Production |
| `ExternalName` | DNS externe | Pointer vers une ressource externe |

---

## Mettre à jour une app — Rolling Update

```bash
# Changer l'image (déclenche un rolling update automatique)
kubectl set image deployment/mon-app mon-app=nginx:1.26

# Voir la progression
kubectl rollout status deployment/mon-app
# Waiting for deployment "mon-app" rollout to finish: 1 out of 3 new replicas have been updated...
# Waiting for deployment "mon-app" rollout to finish: 2 out of 3...
# deployment "mon-app" successfully rolled out

# Voir l'historique
kubectl rollout history deployment/mon-app

# Annuler si problème
kubectl rollout undo deployment/mon-app

# Revenir à une version spécifique
kubectl rollout undo deployment/mon-app --to-revision=2
```

!!! tip "Rolling Update — Comment ça marche"
    K8s crée les nouveaux pods **avant** de supprimer les anciens → zéro downtime.
    Si les nouveaux pods ne démarrent pas correctement, le déploiement se bloque et l'ancienne version continue de tourner.

---

## Namespaces — Isoler les environnements

```bash
# Créer des namespaces
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace prod

# Travailler dans un namespace spécifique
kubectl apply -f deployment.yaml -n dev
kubectl get pods -n dev

# Changer le namespace par défaut (évite de taper -n à chaque fois)
kubectl config set-context --current --namespace=dev
```

---

## ConfigMap — Variables de configuration

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_host: "postgres-service"
  database_port: "5432"
  app_env: "production"
  config.json: |            # Fichier entier en ConfigMap
    {
      "maxConnections": 100,
      "timeout": 30
    }
```

```yaml
# Utiliser dans le Deployment
envFrom:
- configMapRef:
    name: app-config          # Injecte TOUTES les clés comme variables d'env

# Ou fichier monté comme volume
volumes:
- name: config-volume
  configMap:
    name: app-config
volumeMounts:
- name: config-volume
  mountPath: /etc/config
```

!!! success "Checkpoint débutant ✅"
    Tu sais : déployer une app, l'exposer, la mettre à jour, utiliser namespaces et ConfigMaps.
