# Minimal Installation & Configuration Guide  
**Argo Rollouts + Envoy Gateway (Gateway API) + Kustomize**

This guide provides the minimal installation steps and commands required to enable **Gateway API–based traffic routing** with **Argo Rollouts**.

---

## 1. Install Envoy Gateway (Gateway API Controller)

```sh
kubectl create namespace envoy-gateway-system

helm install eg oci://docker.io/envoyproxy/gateway-helm \
  --version v1.6.1 \
  -n envoy-gateway-system \
  --create-namespace
```

**Verify Installation**
```sh
kubectl get pods -n envoy-gateway-system
kubectl get gatewayclass
```

---

## 2. Install Argo Rollouts Controller

```sh
kubectl create namespace argo-rollouts

kubectl apply -n argo-rollouts \
  -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

**Verify**
```sh
kubectl -n argo-rollouts get pods
```

---

## 3. Configure Argo Rollouts GatewayAPI Traffic Router Plugin

**Apply Plugin ConfigMap**
```sh
kubectl apply -f argo-rollout/plugin-configmap.yaml
```

**Apply RBAC**
```sh
kubectl apply -f argo-rollout/gatewayapi-rbac.yaml
```

**Restart Controller**
```sh
kubectl -n argo-rollouts rollout restart deployment/argo-rollouts
```

**Check Plugin Loaded**
```sh
kubectl -n argo-rollouts logs deployment/argo-rollouts --tail=50
```

---

## 4. Apply Production Gateway API Resources

```sh
kubectl apply -k prod/
```

**Verify**
```sh
kubectl -n gateway-system get gateway
kubectl -n prod get httproute -o wide
```

---

## 5. Deploy Rollout and Services (Production)

```sh
kubectl apply -k prod/
```

**Watch Rollout**
```sh
kubectl argo rollouts get rollout vote -n prod --watch
```

---

## 6. Trigger Canary Deployment

**Patch image for new version**
```sh
kubectl -n prod patch rollout vote \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value": "schoolofdevops/vote:v2"}]'
```

**Inspect Traffic Split**
```sh
kubectl -n prod get httproute vote-httproute -o yaml | yq '.spec.rules[0].backendRefs'
```

---

## 7. Rollout Management Commands

**Promote Rollout**
```sh
kubectl argo rollouts promote vote -n prod
```

**Abort Rollout**
```sh
kubectl argo rollouts abort vote -n prod
```

**Retry Rollout**
```sh
kubectl argo rollouts retry vote -n prod
```

---

## 8. Cleanup

```sh
kubectl delete -k prod/
kubectl delete ns argo-rollouts envoy-gateway-system prod
```
