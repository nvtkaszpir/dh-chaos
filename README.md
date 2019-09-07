
# Distributed hackaton chaos-toolkit

## Before you begin and use specific cluster

Create kubernetes cluster with 4 nodes.

```bash
export KUBECONFIG="$(pwd)/.kube/config"
```

## On fresh cluster

Install helm v2.14.3

```bash
kubectl apply -f provision/kubernetes/helm-tiller.rbac.yaml

helm init --service-account tiller --wait

```

Apply labels on the nodes:

```bash
kubectl get nodes
NAME                       STATUS   ROLES   AGE   VERSION
aks-agentpool-32137755-0   Ready    agent   33m   v1.13.10
aks-agentpool-32137755-1   Ready    agent   33m   v1.13.10
aks-agentpool-32137755-2   Ready    agent   33m   v1.13.10
aks-agentpool-32137755-3   Ready    agent   33m   v1.13.10
```

```bash

kubectl label nodes aks-agentpool-32137755-0 app=web
kubectl label nodes aks-agentpool-32137755-1 app=web

kubectl label nodes aks-agentpool-32137755-2 app=mysql

## Deploying app

Using Bitnami Wordpress.

```bash
helm install --name wp-01 stable/wordpress
```

It takes about few minutes due to the way disks are attached to k8s

## Install local depenencies

```bash
pyenv virtualenv chaos
pyenv activate chaos

pip install -r requirements.txt

```

## Look at the app

```bash
kubectl port-forward wp-01-wordpress-7d8bd5468-dw5r9 8080:80
kubectl get endpoints
```
