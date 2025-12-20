# Argo Rollouts + Kustomize Deployment

Minimal repo demonstrating progressive delivery using **Argo Rollouts** with **Kustomize**.  
Base holds shared resources; overlays (e.g., *staging*) control environment-specific image tags and patches.

## Requirements
- Kubernetes cluster (1.20+)  
- `kubectl` + `kustomize`  
- Argo Rollouts CRDs & controller:
```sh
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

