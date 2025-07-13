# Padmini PPCS Apps - Kubernetes Infrastructure GitOps

This repository contains the ArgoCD application definitions and Kubernetes manifests for the Padmini PPCS (Platform Production Control System) cluster infrastructure. It implements a GitOps approach using ArgoCD for automated deployment and management of core platform services.

## 🏗️ Project Overview

This project provides a complete Kubernetes infrastructure stack managed through ArgoCD's app-of-apps pattern. The root application orchestrates the deployment of essential platform components including ingress controllers, certificate management, and cluster-wide resources.

## 📋 Components

### Core Infrastructure Stack

1. **ArgoCD Root Application** (`root-app.yaml`)
   - Implements the app-of-apps pattern
   - Manages all child applications through GitOps
   - Configured for automated sync with self-healing

2. **NGINX Ingress Controller** (`nginx-ingress.yaml`)
   - Kubernetes ingress controller for HTTP/HTTPS traffic routing
   - Deployed as DaemonSet with host networking for optimal performance
   - Version: 4.13.0
   - Resource-optimized configuration (100m CPU, 128Mi memory requests)

3. **Cert-Manager** (`cert-manager-helm.yaml`)
   - Automated TLS certificate management for Kubernetes
   - Version: v1.18.2
   - Integrated with Let's Encrypt for automatic SSL certificate provisioning
   - Custom resource definitions deployed separately for proper initialization order

4. **Certificate CRDs** (`cert-manager-crds.yaml`)
   - Custom Resource Definitions for cert-manager
   - Deployed with high priority (sync-wave: -10) to ensure availability before cert-manager

5. **Cluster Resources** (`cluster-resources.yaml`)
   - Contains cluster-wide configurations and policies
   - Includes Let's Encrypt ClusterIssuer for production SSL certificates
   - Email: khushisoftwareindia@gmail.com for certificate notifications

## 🔄 Deployment Architecture

### Sync Wave Strategy
The applications are deployed in a specific order using ArgoCD sync waves:

1. **Wave -10**: Cert-Manager CRDs (highest priority)
2. **Wave -5**: Cert-Manager application
3. **Wave -1**: Cluster resources (ClusterIssuer, etc.)
4. **Wave 0**: NGINX Ingress Controller (default)

### Repository Structure
```
├── README.md                 # This documentation
├── root-app.yaml            # ArgoCD root application (app-of-apps)
├── argocd-apps/             # ArgoCD application definitions
│   ├── cert-manager-crds.yaml
│   ├── cert-manager-helm.yaml
│   ├── cluster-resources.yaml
│   └── nginx-ingress.yaml
├── cert-manager/            # Cert-manager configuration
│   └── values.yaml
├── cert-manager-crds/       # CRD kustomization
│   └── kustomization.yaml
├── cluster-resources/       # Cluster-wide resources
│   └── cluster-issuer.yaml
└── nginx-ingress/          # NGINX ingress configuration
    └── values.yaml
```

## ⚙️ Configuration Details

### Resource Management
- **Cert-Manager**: 100m CPU / 128Mi memory (requests), 250m CPU / 256Mi memory (limits)
- **NGINX Ingress**: 100m CPU / 128Mi memory (requests), 200m CPU / 256Mi memory (limits)
- Optimized for production workloads with appropriate resource constraints

### Security Features
- Automated TLS certificate management via Let's Encrypt
- HTTP01 challenge solver using NGINX ingress
- Production-grade ACME server configuration

### High Availability
- NGINX deployed as DaemonSet for node-level redundancy
- ArgoCD automated sync with self-healing capabilities
- Proper resource lifecycle management with finalizers

## 🚀 Quick Start

1. **Prerequisites**:
   - Kubernetes cluster with ArgoCD installed
   - Access to deploy applications in the `argocd` namespace

2. **Deploy the stack**:
   ```bash
   kubectl apply -f root-app.yaml
   ```

3. **Monitor deployment**:
   - Check ArgoCD UI for application sync status
   - Verify all applications are healthy and synced

## 🔧 Customization

### Modifying Configurations
- Update `values.yaml` files in respective component directories
- Modify resource limits/requests based on cluster capacity
- Adjust ingress controller settings for specific networking requirements

### Adding New Applications
1. Create application definition in `argocd-apps/`
2. Configure appropriate sync waves and policies
3. ArgoCD will automatically detect and deploy new applications

## 📊 Monitoring & Maintenance

- Applications are configured with automated sync and self-healing
- Monitor ArgoCD dashboard for deployment status
- Check application health through ArgoCD CLI or UI
- Certificate renewal is handled automatically by cert-manager

## 🏷️ Labels & Annotations

- **Cluster Name**: padmini (applied to root application)
- **Sync Waves**: Used for deployment ordering
- **Finalizers**: Ensure proper resource cleanup

This infrastructure foundation provides a robust, scalable, and maintainable platform for deploying additional applications and services in the Padmini PPCS environment.