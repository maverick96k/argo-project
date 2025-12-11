Minimal Installation & Configuration Guide
Argo Rollouts + Envoy Gateway (Gateway API) + Kustomize

This document provides the minimal required installation steps and configuration commands to enable Gateway API based traffic routing with Argo Rollouts.


1. Install Envoy Gateway (Gateway API Controller)

kubectl create namespace envoy-gateway-system

helm install eg oci://docker.io/envoyproxy/gateway-helm \
  --version v1.6.1 \
  -n envoy-gateway-system \
  --create-namespace

Verify:
kubectl get pods -n envoy-gateway-system
kubectl get gatewayclass



2. Install Argo Rollouts Controller

kubectl create namespace argo-rollouts

kubectl apply -n argo-rollouts \
  -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

Verify:
kubectl -n argo-rollouts get pods



3. Configure Argo Rollouts GatewayAPI Traffic Router Plugin

Apply plugin configmap:
kubectl apply -f argo-rollout/plugin-configmap.yaml

Apply RBAC:
kubectl apply -f argo-rollout/gatewayapi-rbac.yaml

Restart controller:
kubectl -n argo-rollouts rollout restart deployment/argo-rollouts

Check plugin loaded:
kubectl -n argo-rollouts logs deployment/argo-rollouts --tail=50



4. Apply Production Gateway API Resources

kubectl apply -k prod/

Verify:
kubectl -n gateway-system get gateway
kubectl -n prod get httproute -o wide



5. Deploy Rollout and Services (Production)

kubectl apply -k prod/

Watch rollout:
kubectl argo rollouts get rollout vote -n prod --watch



6. Trigger Canary Deployment

Update image:
kubectl -n prod patch rollout vote \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value": "schoolofdevops/vote:v2"}]'

Inspect traffic split:
kubectl -n prod get httproute vote-httproute -o yaml | yq '.spec.rules[0].backendRefs'



7. Rollout Management Commands

Promote:
kubectl argo rollouts promote vote -n prod

Abort:
kubectl argo rollouts abort vote -n prod

Retry:
kubectl argo rollouts retry vote -n prod



8. Cleanup

kubectl delete -k prod/
kubectl delete ns argo-rollouts envoy-gateway-system prod

