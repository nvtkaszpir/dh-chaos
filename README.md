
# Distributed hackaton chaos-toolkit

## Before you begin and use specific cluster

```bash
export KUBECONFIG="$(pwd)/.kube/config"
```

## On fresh cluster

Install helm v2.14.3

```bash
kubectl apply -f provision/kubernetes/helm-tiller.rbac.yaml

helm init --service-account tiller --wait

```

## Add helm azure repo

```bash
helm repo update
```

## Deploying app

Using Bitnami Wordpress.

```bash
helm install --name wp-01 stable/wordpress
```

It takes about few minutes due to the way disks are attached to k8s

