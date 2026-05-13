---
id: kubernetes
title: Kubernetes Deployment
sidebar_label: Kubernetes
sidebar_position: 2
---

# Kubernetes Deployment

Clawboard can be deployed to Kubernetes using the manifests included in `api/k8s`.

## Repository structure

The monorepo includes Kubernetes manifests under `api/k8s/` (Kustomize layout):

```
k8s/
├── base/
│   ├── kustomization.yaml
│   ├── namespace.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── secret.template.yaml  ← copy to secret.yaml, fill in values
│   └── configmap.yaml
└── overlays/
    └── production/
```

## Deploying

### 1. Configure secrets

```bash
cp k8s/base/secret.template.yaml k8s/base/secret.yaml
```

Edit `secret.yaml` and fill in your base64-encoded values:

```bash
# Encode a value
echo -n "your-value" | base64
```

Required secrets:

```yaml
stringData:
  DB_PASSWORD: 'your-db-password'
  JWT_SECRET: 'your-jwt-secret'
  CORS_ORIGIN: 'https://your-dashboard.example.com'
```

### 2. Apply the manifests

```bash
kubectl apply -k k8s/base
```

Or for a specific overlay:

```bash
kubectl apply -k k8s/overlays/production
```

### 3. Verify

```bash
kubectl get pods -n clawboard
kubectl get svc -n clawboard

# Check API health
kubectl port-forward -n clawboard svc/clawboard-api 3000:3000
curl http://localhost:3000/health
```

## Ingress

Configure an ingress to expose the API and dashboard publicly. Example with nginx ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: clawboard
  namespace: clawboard
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: 'true'
spec:
  rules:
    - host: api-clawboard.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: clawboard-api
                port:
                  number: 3000
    - host: clawboard.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: clawboard-web
                port:
                  number: 80
  tls:
    - hosts:
        - api-clawboard.example.com
        - clawboard.example.com
      secretName: clawboard-tls
```

## Resource recommendations

Use the burstable resource strategy — low requests to unblock the scheduler, high limits to allow
bursting:

```yaml
resources:
  requests:
    cpu: '25m'
    memory: '64Mi'
  limits:
    cpu: '500m'
    memory: '512Mi'
```

## Database

For production Kubernetes deployments, consider using a managed PostgreSQL service (e.g. Cloud SQL,
RDS, Supabase) rather than running PostgreSQL in the cluster. This simplifies backups, scaling, and
maintenance.

If running PostgreSQL in the cluster, use a StatefulSet with a persistent volume:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 1
  template:
    spec:
      containers:
        - name: postgres
          image: postgres:15
          env:
            - name: POSTGRES_DB
              value: clawboard
            - name: POSTGRES_USER
              value: clawboard
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: clawboard-secrets
                  key: DB_PASSWORD
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ['ReadWriteOnce']
        resources:
          requests:
            storage: 10Gi
```

## OpenClaw integration in Kubernetes

If OpenClaw also runs in Kubernetes, use in-cluster service DNS names for the integration:

```yaml
# In clawboard-api secret/configmap
OPENCLAW_WORKSPACE_URL: 'http://openclaw-workspace.openclaw-personal.svc.cluster.local:18780'
OPENCLAW_GATEWAY_URL: 'http://openclaw.openclaw-personal.svc.cluster.local:18789'
```

See [OpenClaw on Kubernetes](../openclaw/kubernetes) for the full guide.

## Updating

```bash
# Update the image tag in your deployment
kubectl set image deployment/clawboard-api api=ghcr.io/moltar-forge/clawboard-api:v1.2.3 -n clawboard

# Or apply updated manifests
kubectl apply -k k8s/base
```

Migrations run automatically on pod startup.
