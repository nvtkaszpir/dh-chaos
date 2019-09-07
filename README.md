
# Distributed hackaton chaos-toolkit

## Before you begin and use specific cluster

```bash
export KUBECONFIG="$(pwd)/.kube/config"
```

## On fresh cluster

Install helm:

```bash
kubectl apply -f provision/kubernetes/helm-tiller.rbac.yaml

helm init --service-account tiller --wait

```

## Add helm azure repo

```bash
helm repo add azure https://kubernetescharts.blob.core.windows.net/azure
```

## Deploying app

```bash
helm install --name wp-01 azure/wordpress
```
