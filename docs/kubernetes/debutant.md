# 🟢 Kubernetes — Débutant

## Les commandes de base

```bash
kubectl get pods              # Lister les pods
kubectl get deployments       # Lister les deployments
kubectl get services          # Lister les services
kubectl get all               # Tout voir

kubectl describe pod mon-pod  # Détails d'un pod
kubectl logs mon-pod          # Logs d'un pod
kubectl exec -it mon-pod -- bash  # Shell dans un pod
```

---

## Ton premier Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mon-app
spec:
  replicas: 3          # 3 instances de l'app
  selector:
    matchLabels:
      app: mon-app
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
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
```

```bash
kubectl apply -f deployment.yaml  # Créer/mettre à jour
kubectl get pods                  # Voir les 3 pods créés
```

---

## Exposer l'app avec un Service

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mon-app-service
spec:
  selector:
    app: mon-app       # Connecte au deployment via le label
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP      # Interne au cluster
  # type: LoadBalancer # Exposé sur internet
  # type: NodePort     # Exposé sur un port du nœud
```

---

## Les types de Service

| Type | Accès | Usage |
|------|-------|-------|
| `ClusterIP` | Interne seulement | Communication entre services |
| `NodePort` | IP du nœud + port | Dev/test |
| `LoadBalancer` | IP publique | Production |

---

## Mettre à jour une app (sans downtime)

```bash
# Changer l'image (rolling update automatique)
kubectl set image deployment/mon-app mon-app=nginx:1.26

# Voir la progression
kubectl rollout status deployment/mon-app

# Annuler si ça se passe mal
kubectl rollout undo deployment/mon-app
```

---

## Namespaces — Isoler les environnements

```bash
# Créer des namespaces
kubectl create namespace dev
kubectl create namespace prod

# Déployer dans un namespace
kubectl apply -f deployment.yaml -n dev

# Voir les ressources d'un namespace
kubectl get all -n dev
```

!!! success "Checkpoint débutant ✅"
    Tu sais déployer une app, l'exposer, la mettre à jour et utiliser les namespaces.
