# 🟡 Kubernetes — Intermédiaire

## ConfigMap et Secret

=== "ConfigMap (données non-sensibles)"
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: app-config
    data:
      DATABASE_HOST: "postgres-service"
      APP_ENV: "production"
      MAX_CONNECTIONS: "100"
    ```

=== "Secret (données sensibles)"
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: app-secrets
    type: Opaque
    stringData:                    # K8s encode en base64 automatiquement
      DATABASE_PASSWORD: "MonMotDePasse"
      API_KEY: "sk-abc123"
    ```

Utiliser dans un Deployment :
```yaml
spec:
  containers:
  - name: mon-app
    envFrom:
    - configMapRef:
        name: app-config
    - secretRef:
        name: app-secrets
```

---

## Ingress — Routage HTTP/S

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mon-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: monapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

---

## HPA — Autoscaling automatique

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: mon-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: mon-app
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70   # Scale si CPU > 70%
```

---

## Probes — Vérifier la santé de l'app

```yaml
containers:
- name: mon-app
  livenessProbe:            # K8s redémarre si ça échoue
    httpGet:
      path: /health
      port: 8080
    initialDelaySeconds: 30
    periodSeconds: 10

  readinessProbe:           # K8s enlève du load balancer si ça échoue
    httpGet:
      path: /ready
      port: 8080
    initialDelaySeconds: 5
    periodSeconds: 5
```

| Probe | Action si échec |
|-------|----------------|
| `livenessProbe` | Redémarre le container |
| `readinessProbe` | Retire du trafic (mais ne redémarre pas) |
| `startupProbe` | Attend que l'app démarre avant les autres probes |

---

## Helm — Le gestionnaire de packages K8s

```bash
# Installer Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Ajouter un repo et installer une app
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx

# Voir les releases
helm list

# Mettre à jour
helm upgrade nginx-ingress ingress-nginx/ingress-nginx

# Désinstaller
helm uninstall nginx-ingress
```

!!! success "Checkpoint intermédiaire ✅"
    Tu gères : ConfigMap/Secret, Ingress, HPA, Probes et Helm.
