# 🔴 Kubernetes — Avancé

## RBAC — Contrôle d'accès

```yaml
# Role — Permissions dans un namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: dev-role
  namespace: dev
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "pods"]
  verbs: ["get", "list", "watch", "create", "update"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]

---
# RoleBinding — Attribuer le role à un utilisateur
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-binding
  namespace: dev
subjects:
- kind: User
  name: leo@example.com
roleRef:
  kind: Role
  name: dev-role
  apiGroup: rbac.authorization.k8s.io
```

---

## NetworkPolicy — Firewall entre pods

```yaml
# Autoriser uniquement le frontend à parler à l'API
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-network-policy
spec:
  podSelector:
    matchLabels:
      app: api
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

---

## PersistentVolume — Données persistantes

```yaml
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard

---
# Utiliser dans un pod
volumes:
- name: postgres-data
  persistentVolumeClaim:
    claimName: postgres-pvc

volumeMounts:
- name: postgres-data
  mountPath: /var/lib/postgresql/data
```

---

## Operators — Automatiser la gestion d'apps complexes

```bash
# Exemple : déployer PostgreSQL avec l'opérateur CloudNativePG
kubectl apply -f https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/main/releases/cnpg-1.22.0.yaml

# Puis créer un cluster PostgreSQL en quelques lignes YAML
kubectl apply -f - <<EOF
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: postgres-cluster
spec:
  instances: 3
  storage:
    size: 20Gi
EOF
```

---

## Debugging avancé

```bash
# Pod qui ne démarre pas
kubectl describe pod mon-pod     # Voir les Events
kubectl logs mon-pod --previous  # Logs du container précédent (avant crash)

# Port-forward pour debugger en local
kubectl port-forward pod/mon-pod 8080:80
kubectl port-forward service/mon-service 8080:80

# Lancer un pod de debug temporaire
kubectl run debug --image=busybox -it --rm -- sh

# Voir la consommation de ressources
kubectl top pods
kubectl top nodes
```

---

## Bonnes pratiques

| Pratique | Pourquoi |
|----------|----------|
| Toujours définir `requests` et `limits` | Évite qu'un pod consomme tout |
| Utiliser des `readinessProbe` | Évite le trafic vers des pods non prêts |
| Namespaces par équipe/environnement | Isolation et RBAC simplifié |
| Eviter `latest` comme tag d'image | Déploiements non reproductibles |
| GitOps avec Argo CD ou Flux | Infra déclarative et auditée |
