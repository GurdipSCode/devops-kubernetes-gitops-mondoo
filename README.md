# devops-kubernetes-gitops-mondoo

[![Mondoo](https://img.shields.io/badge/Mondoo-Security-00A3E0?style=flat-square&logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOCAYAAAAfSC3RAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAkSURBVHgB7cxBDQAgAMAwjP7/aKNhLoDt5wcAAAAAAAAAALwGK7wB8bYAAAAASUVORK5CYII=)](https://mondoo.com)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-EF7B4D?style=flat-square&logo=argo&logoColor=white)](https://argo-cd.readthedocs.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.24+-326CE5?style=flat-square&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Helm](https://img.shields.io/badge/Helm-v3-0F1689?style=flat-square&logo=helm&logoColor=white)](https://helm.sh/)
[![License](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](LICENSE)
[![GitOps](https://img.shields.io/badge/GitOps-Enabled-success?style=flat-square)](https://www.gitops.tech/)
[![Security](https://img.shields.io/badge/Security-Scanning-red?style=flat-square&logo=security&logoColor=white)](https://mondoo.com)
[![Compliance](https://img.shields.io/badge/Compliance-CIS%20%7C%20NIST-blue?style=flat-square)](https://mondoo.com)

Deploy [Mondoo](https://mondoo.com) security and compliance scanning in your Kubernetes cluster using [ArgoCD](https://argo-cd.readthedocs.io/) for GitOps-based continuous deployment.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Architecture](#architecture)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Configuration](#configuration)
- [ArgoCD Application](#argocd-application)
- [Usage](#usage)
- [Monitoring](#monitoring)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)
- [Contributing](#contributing)

## 🌟 Overview

This repository contains GitOps configurations to deploy Mondoo security scanning platform to Kubernetes using ArgoCD. Mondoo provides:

- **Container Security Scanning** - Scan container images for vulnerabilities
- **Kubernetes Security Posture** - Assess cluster security configurations
- **Compliance Scanning** - CIS, NIST, PCI-DSS, SOC 2, and more
- **Policy as Code** - Define and enforce security policies
- **Continuous Monitoring** - Real-time security and compliance monitoring

## ✨ Features

- ✅ **GitOps-based deployment** using ArgoCD
- ✅ **Automated sync** from Git repository
- ✅ **Multi-environment support** (dev, staging, production)
- ✅ **Helm-based** installation with customizable values
- ✅ **Secret management** with Sealed Secrets or External Secrets
- ✅ **Health checks** and status monitoring
- ✅ **Automated updates** with image updater
- ✅ **Policy enforcement** via OPA Gatekeeper integration

## 📦 Prerequisites

Before deploying Mondoo via ArgoCD, ensure you have:

| Requirement | Version | Description |
|-------------|---------|-------------|
| **Kubernetes** | 1.24+ | Target cluster |
| **ArgoCD** | 2.8+ | GitOps deployment tool |
| **Helm** | 3.0+ | Package manager (for local testing) |
| **kubectl** | 1.24+ | Kubernetes CLI |
| **Mondoo Account** | - | Free account at [console.mondoo.com](https://console.mondoo.com) |

### Create Mondoo Account

1. Sign up at [console.mondoo.com](https://console.mondoo.com)
2. Create a new Space
3. Generate a Service Account token
4. Download the credentials JSON file

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         Git Repository                        │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  manifests/                                          │   │
│  │  ├── base/                                           │   │
│  │  │   ├── namespace.yaml                              │   │
│  │  │   ├── mondoo-secret.yaml (sealed)                 │   │
│  │  │   └── mondoo-values.yaml                          │   │
│  │  └── overlays/                                       │   │
│  │      ├── dev/                                        │   │
│  │      ├── staging/                                    │   │
│  │      └── production/                                 │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↓ (ArgoCD syncs)
┌─────────────────────────────────────────────────────────────┐
│                      ArgoCD                                   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Application: mondoo                                 │   │
│  │  - Auto-sync: enabled                                │   │
│  │  - Self-heal: enabled                                │   │
│  │  - Prune: enabled                                    │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↓ (deploys to)
┌─────────────────────────────────────────────────────────────┐
│                   Kubernetes Cluster                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Namespace: mondoo                                   │   │
│  │  ├── mondoo-operator (Deployment)                    │   │
│  │  ├── mondoo-client (DaemonSet)                       │   │
│  │  ├── mondoo-admission-controller (Deployment)        │   │
│  │  └── mondoo-config (Secret)                          │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↓ (reports to)
┌─────────────────────────────────────────────────────────────┐
│                    Mondoo Console                             │
│           https://console.mondoo.com                          │
└─────────────────────────────────────────────────────────────┘
```

## 🚀 Quick Start

### 1. Clone This Repository

```bash
git clone https://github.com/your-org/mondoo-argocd.git
cd mondoo-argocd
```

### 2. Create Mondoo Secret

```bash
# Create namespace
kubectl create namespace mondoo

# Create secret from Mondoo credentials
kubectl create secret generic mondoo-client \
  --from-file=config=/path/to/mondoo-credentials.json \
  -n mondoo

# Seal the secret (if using Sealed Secrets)
kubeseal --format=yaml --cert=pub-cert.pem \
  < mondoo-secret.yaml > mondoo-sealed-secret.yaml
```

### 3. Deploy ArgoCD Application

```bash
kubectl apply -f argocd/mondoo-application.yaml
```

### 4. Verify Deployment

```bash
# Check ArgoCD sync status
argocd app get mondoo

# Check Mondoo pods
kubectl get pods -n mondoo

# View Mondoo operator logs
kubectl logs -n mondoo -l app=mondoo-operator
```

## 📥 Installation

### Directory Structure

```
mondoo-argocd/
├── argocd/
│   ├── mondoo-application.yaml          # ArgoCD Application manifest
│   ├── mondoo-appproject.yaml           # ArgoCD AppProject (optional)
│   └── image-updater-config.yaml        # Image updater config
├── manifests/
│   ├── base/
│   │   ├── kustomization.yaml
│   │   ├── namespace.yaml
│   │   ├── mondoo-values.yaml           # Helm values
│   │   └── mondoo-secret-sealed.yaml    # Sealed secret
│   └── overlays/
│       ├── dev/
│       │   ├── kustomization.yaml
│       │   └── mondoo-values.yaml
│       ├── staging/
│       │   ├── kustomization.yaml
│       │   └── mondoo-values.yaml
│       └── production/
│           ├── kustomization.yaml
│           └── mondoo-values.yaml
├── policies/
│   └── mondoo-policies.yaml             # Custom Mondoo policies
└── README.md
```

### Step-by-Step Installation

#### Step 1: Create Namespace

```yaml
# manifests/base/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: mondoo
  labels:
    name: mondoo
    app.kubernetes.io/name: mondoo
    app.kubernetes.io/managed-by: argocd
```

#### Step 2: Create Mondoo Secret

**Option A: Using Sealed Secrets (Recommended)**

```bash
# 1. Create regular secret
kubectl create secret generic mondoo-client \
  --from-file=config=mondoo-credentials.json \
  --dry-run=client -o yaml > mondoo-secret.yaml

# 2. Seal the secret
kubeseal --format=yaml --cert=pub-cert.pem \
  < mondoo-secret.yaml > manifests/base/mondoo-secret-sealed.yaml

# 3. Commit sealed secret to Git
git add manifests/base/mondoo-secret-sealed.yaml
git commit -m "Add sealed Mondoo credentials"
git push
```

**Option B: Using External Secrets Operator**

```yaml
# manifests/base/mondoo-external-secret.yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: mondoo-client
  namespace: mondoo
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: mondoo-client
    creationPolicy: Owner
  data:
    - secretKey: config
      remoteRef:
        key: secret/mondoo/credentials
        property: config
```

#### Step 3: Configure Helm Values

```yaml
# manifests/base/mondoo-values.yaml
mondoo:
  # Mondoo operator configuration
  operator:
    image:
      repository: mondoo/mondoo-operator
      tag: latest
      pullPolicy: IfNotPresent
    
    resources:
      limits:
        cpu: 500m
        memory: 512Mi
      requests:
        cpu: 100m
        memory: 128Mi

  # Mondoo client (DaemonSet)
  client:
    enable: true
    image:
      repository: mondoo/mondoo
      tag: latest
    
    resources:
      limits:
        cpu: 200m
        memory: 256Mi
      requests:
        cpu: 50m
        memory: 64Mi
    
    # Scan configuration
    scan:
      schedule: "0 */6 * * *"  # Every 6 hours
      containers: true
      nodes: true
      workloads: true

  # Admission controller (optional)
  admission:
    enable: true
    mode: "enforce"  # enforce, monitor, or disabled
    
    replicas: 2
    
    resources:
      limits:
        cpu: 300m
        memory: 384Mi
      requests:
        cpu: 100m
        memory: 128Mi
    
    # Policy enforcement
    policies:
      - critical-vulnerabilities
      - cis-kubernetes
      - pod-security-standards

  # Integration settings
  integration:
    kubernetes:
      enabled: true
      scanInterval: 6h
    
    # Report findings
    reporting:
      webhook:
        enabled: false
        url: ""
      slack:
        enabled: false
        webhook: ""

  # RBAC
  rbac:
    create: true
  
  # Service Account
  serviceAccount:
    create: true
    name: mondoo

# Additional Kubernetes resources
nodeSelector: {}
tolerations: []
affinity: {}
```

#### Step 4: Create Kustomization

```yaml
# manifests/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: mondoo

resources:
  - namespace.yaml
  - mondoo-secret-sealed.yaml

helmCharts:
  - name: mondoo-operator
    repo: https://mondoohq.github.io/mondoo-operator
    version: 1.14.0
    releaseName: mondoo
    namespace: mondoo
    valuesFile: mondoo-values.yaml
```

#### Step 5: Create Environment Overlays

**Development Environment:**

```yaml
# manifests/overlays/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: mondoo

bases:
  - ../../base

patchesStrategicMerge:
  - mondoo-values.yaml
```

```yaml
# manifests/overlays/dev/mondoo-values.yaml
mondoo:
  admission:
    mode: "monitor"  # Don't enforce in dev
  
  client:
    scan:
      schedule: "0 */12 * * *"  # Less frequent in dev
```

**Production Environment:**

```yaml
# manifests/overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: mondoo

bases:
  - ../../base

patchesStrategicMerge:
  - mondoo-values.yaml
```

```yaml
# manifests/overlays/production/mondoo-values.yaml
mondoo:
  admission:
    mode: "enforce"  # Enforce in production
    replicas: 3  # High availability
  
  client:
    scan:
      schedule: "0 */4 * * *"  # More frequent in prod
    
    resources:
      limits:
        cpu: 500m
        memory: 512Mi
```

## 🔧 Configuration

### ArgoCD Application

Create the ArgoCD Application manifest:

```yaml
# argocd/mondoo-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: mondoo
  namespace: argocd
  labels:
    app: mondoo
    environment: production
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  
  # Source repository
  source:
    repoURL: https://github.com/your-org/mondoo-argocd.git
    targetRevision: main
    path: manifests/overlays/production
  
  # Destination cluster
  destination:
    server: https://kubernetes.default.svc
    namespace: mondoo
  
  # Sync policy
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
    
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  
  # Health checks
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
```

### Multi-Environment Setup

```yaml
# argocd/mondoo-application-dev.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: mondoo-dev
  namespace: argocd
spec:
  project: mondoo
  source:
    repoURL: https://github.com/your-org/mondoo-argocd.git
    targetRevision: develop
    path: manifests/overlays/dev
  destination:
    server: https://dev-cluster.example.com
    namespace: mondoo
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

```yaml
# argocd/mondoo-application-prod.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: mondoo-prod
  namespace: argocd
spec:
  project: mondoo
  source:
    repoURL: https://github.com/your-org/mondoo-argocd.git
    targetRevision: main
    path: manifests/overlays/production
  destination:
    server: https://prod-cluster.example.com
    namespace: mondoo
  syncPolicy:
    automated:
      prune: true
      selfHeal: false  # Manual sync in production
```

### ArgoCD AppProject (Optional)

```yaml
# argocd/mondoo-appproject.yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: mondoo
  namespace: argocd
spec:
  description: Mondoo Security Platform
  
  # Source repositories
  sourceRepos:
    - https://github.com/your-org/mondoo-argocd.git
    - https://mondoohq.github.io/mondoo-operator
  
  # Destination clusters
  destinations:
    - namespace: mondoo
      server: https://kubernetes.default.svc
    - namespace: mondoo
      server: https://dev-cluster.example.com
    - namespace: mondoo
      server: https://prod-cluster.example.com
  
  # Allowed cluster resources
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
    - group: ''
      kind: ClusterRole
    - group: ''
      kind: ClusterRoleBinding
    - group: 'admissionregistration.k8s.io'
      kind: ValidatingWebhookConfiguration
    - group: 'admissionregistration.k8s.io'
      kind: MutatingWebhookConfiguration
  
  # Namespace resources
  namespaceResourceWhitelist:
    - group: '*'
      kind: '*'
  
  # RBAC roles
  roles:
    - name: read-only
      description: Read-only access
      policies:
        - p, proj:mondoo:read-only, applications, get, mondoo/*, allow
    
    - name: admin
      description: Admin access
      policies:
        - p, proj:mondoo:admin, applications, *, mondoo/*, allow
```

## 🎯 Usage

### Deploy Mondoo

```bash
# Apply ArgoCD Application
kubectl apply -f argocd/mondoo-application.yaml

# Watch deployment
argocd app get mondoo --watch

# Or use kubectl
kubectl get applications -n argocd -w
```

### Sync Application

```bash
# Manual sync
argocd app sync mondoo

# Sync with prune
argocd app sync mondoo --prune

# Force sync
argocd app sync mondoo --force
```

### View Application Status

```bash
# Get application details
argocd app get mondoo

# View sync history
argocd app history mondoo

# View application tree
argocd app tree mondoo
```

### Check Mondoo Status

```bash
# Check all Mondoo resources
kubectl get all -n mondoo

# Check operator logs
kubectl logs -n mondoo -l app=mondoo-operator -f

# Check client pods (DaemonSet)
kubectl get pods -n mondoo -l app=mondoo-client

# Check admission controller
kubectl logs -n mondoo -l app=mondoo-admission
```

### View Scan Results

```bash
# Get scan reports
kubectl get mondooauditconfigs -n mondoo

# View specific scan
kubectl describe mondooauditconfig cluster-scan -n mondoo

# Check for vulnerabilities
kubectl get vulnerabilities -n mondoo
```

## 📊 Monitoring

### Health Checks

ArgoCD monitors these resources:

```yaml
# Health assessment rules
spec:
  health:
    mondooauditconfig:
      # Custom health check for Mondoo CRDs
      lua: |
        hs = {}
        if obj.status ~= nil then
          if obj.status.conditions ~= nil then
            for i, condition in ipairs(obj.status.conditions) do
              if condition.type == "Available" and condition.status == "True" then
                hs.status = "Healthy"
                hs.message = "Mondoo is scanning"
                return hs
              end
            end
          end
        end
        hs.status = "Progressing"
        hs.message = "Waiting for scan to complete"
        return hs
```

### Prometheus Metrics

```yaml
# ServiceMonitor for Prometheus
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: mondoo-operator
  namespace: mondoo
spec:
  selector:
    matchLabels:
      app: mondoo-operator
  endpoints:
    - port: metrics
      interval: 30s
      path: /metrics
```

### Grafana Dashboard

Import Mondoo dashboard:
```bash
# Dashboard ID: TBD (create custom dashboard)
# Monitor:
# - Scan success rate
# - Vulnerabilities found
# - Policy violations
# - Scan duration
```

## 🔍 Troubleshooting

### Common Issues

#### 1. Application Not Syncing

```bash
# Check application status
argocd app get mondoo

# View sync errors
argocd app sync mondoo --dry-run

# Check for differences
argocd app diff mondoo
```

#### 2. Pods Not Starting

```bash
# Check pod status
kubectl get pods -n mondoo

# View pod events
kubectl describe pod <pod-name> -n mondoo

# Check logs
kubectl logs <pod-name> -n mondoo
```

#### 3. Mondoo Client Authentication Failed

```bash
# Verify secret exists
kubectl get secret mondoo-client -n mondoo

# Check secret content
kubectl get secret mondoo-client -n mondoo -o jsonpath='{.data.config}' | base64 -d

# Recreate secret
kubectl delete secret mondoo-client -n mondoo
kubectl create secret generic mondoo-client \
  --from-file=config=mondoo-credentials.json \
  -n mondoo
```

#### 4. Admission Controller Webhook Issues

```bash
# Check webhook configuration
kubectl get validatingwebhookconfigurations | grep mondoo

# View webhook logs
kubectl logs -n mondoo -l app=mondoo-admission -f

# Test webhook
kubectl run test-pod --image=nginx --dry-run=server
```

### Debug Commands

```bash
# ArgoCD debugging
argocd app manifests mondoo               # View rendered manifests
argocd app logs mondoo                    # View application logs
argocd app events mondoo                  # View application events

# Kubernetes debugging
kubectl get events -n mondoo --sort-by='.lastTimestamp'
kubectl top pods -n mondoo
kubectl describe mondooauditconfig -n mondoo

# Helm debugging (for local testing)
helm template mondoo mondoo-operator/mondoo-operator \
  -f manifests/base/mondoo-values.yaml \
  --namespace mondoo
```

## 🔒 Security Considerations

### Secret Management

1. **Never commit plaintext secrets** to Git
2. **Use Sealed Secrets or External Secrets Operator**
3. **Rotate Mondoo credentials** regularly
4. **Limit secret access** with RBAC

```yaml
# Example: Rotate secret
apiVersion: v1
kind: Secret
metadata:
  name: mondoo-client
  namespace: mondoo
  annotations:
    sealedsecrets.bitnami.com/managed: "true"
    rotation-date: "2024-01-01"
type: Opaque
data:
  config: <base64-encoded-new-credentials>
```

### RBAC

```yaml
# Limit ArgoCD access to Mondoo namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: mondoo-manager
  namespace: mondoo
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: mondoo-manager-binding
  namespace: mondoo
subjects:
  - kind: ServiceAccount
    name: argocd-application-controller
    namespace: argocd
roleRef:
  kind: Role
  name: mondoo-manager
  apiGroup: rbac.authorization.k8s.io
```

### Network Policies

```yaml
# Restrict Mondoo network access
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mondoo-netpol
  namespace: mondoo
spec:
  podSelector:
    matchLabels:
      app: mondoo
  policyTypes:
    - Ingress
    - Egress
  egress:
    # Allow DNS
    - to:
        - namespaceSelector:
            matchLabels:
              name: kube-system
      ports:
        - protocol: UDP
          port: 53
    # Allow Mondoo API
    - to:
        - podSelector: {}
      ports:
        - protocol: TCP
          port: 443
```

## 📈 Best Practices

### GitOps Workflow

1. **Make changes in Git first**
2. **Create Pull Requests** for reviews
3. **Use branch protection** for production
4. **Enable auto-sync** for non-production
5. **Manual sync** for production deployments

### Version Pinning

```yaml
# Pin Helm chart version
helmCharts:
  - name: mondoo-operator
    version: 1.14.0  # Specific version, not 'latest'
```

### Progressive Rollout

```yaml
# Use ArgoCD Progressive Sync
spec:
  syncPolicy:
    automated: null  # Disable auto-sync
    syncOptions:
      - CreateNamespace=true
  
  # Manual approval for production
  # Use ArgoCD Rollouts for canary deployments
```

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📚 Resources

### Mondoo Documentation
- [Mondoo Docs](https://mondoo.com/docs/)
- [Mondoo Operator GitHub](https://github.com/mondoohq/mondoo-operator)
- [Mondoo Helm Chart](https://github.com/mondoohq/mondoo-operator/tree/main/charts)

### ArgoCD Documentation
- [ArgoCD Docs](https://argo-cd.readthedocs.io/)
- [ArgoCD Best Practices](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/)
- [GitOps Guide](https://www.gitops.tech/)

### Related Projects
- [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)
- [External Secrets Operator](https://external-secrets.io/)
- [ArgoCD Image Updater](https://argocd-image-updater.readthedocs.io/)

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Mondoo Team](https://mondoo.com) for the security platform
- [ArgoCD Team](https://argoproj.github.io/) for the GitOps tool
- Community contributors

## 📞 Support

- **Mondoo Support**: [support@mondoo.com](mailto:support@mondoo.com)
- **Community Slack**: [Mondoo Slack](https://mondoo.link/slack)
- **Issues**: [GitHub Issues](https://github.com/your-org/mondoo-argocd/issues)

---

**Made with ❤️ for Kubernetes Security**

[![Star on GitHub](https://img.shields.io/github/stars/your-org/mondoo-argocd?style=social)](https://github.com/your-org/mondoo-argocd)
[![Follow on Twitter](https://img.shields.io/twitter/follow/mondoohq?style=social)](https://twitter.com/mondoohq)
