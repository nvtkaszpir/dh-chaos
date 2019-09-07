# Before you begin and use specific cluster
```bash
export KUBECONFIG="$(pwd)/.kube/config"
```

# On fresh cluster
- install helm:

```bash
kubectl apply -f provision/kubernetes/helm-tiller.rbac.yaml

helm init --service-account tiller --wait

```

# Deploying app


