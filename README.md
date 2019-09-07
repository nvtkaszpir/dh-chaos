
# Distributed hackaton chaos-toolkit

Use case:

- get to know managed Kubernetes on Microsoft Azure
- use chaos-toolkit to play around

## Before you begin and use specific cluster

Create kubernetes cluster with 4 nodes.

```bash
export KUBECONFIG="$(pwd)/.kube/config"
```

## On fresh cluster

Install helm [v2.14.3](https://github.com/helm/helm/releases/tag/v2.14.3) 

Then we need to apply RBAC on cluster and deploy helm:

```bash
kubectl apply -f provision/kubernetes/helm-tiller.rbac.yaml

helm init --service-account tiller --wait

```

### Optional - apply labels on the nodes

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

kubectl label node aks-agentpool-32137755-3 app=drain-me --overwrite
```

## Add azurefile storage provider

If you are using Azure k8s then we need read-write-many storageclass to allow
multiple pods to write to storage:

```bash
kubectl apply -f provision/kubernetes/azure-pvc-roles.yaml
kubectl apply -f provision/kubernetes/azure-file-sc.yaml
```

## Deploying app

Using Bitnami Wordpress:

```bash
helm install --name wp-02 stable/wordpress -f wordpress-azurefile.yaml
```

It takes about few minutes due to the way disks are attached to k8s.

## Look at the app

```bash
kubectl port-forward service/wp-02-wordpress 80:80 8081:80
kubectl get endpoints
kubectl get svc
```

Notice that Azure managed kubernetes is exposing app publicly to the internet.

## Install local dependencies for chaos toolkit

```bash
pyenv virtualenv chaos
pyenv activate chaos

pip install -r requirements.txt

```

## Using chaos toolkit

Remember to export `KUBECONFIG`.

First, run chaos discovery to fetch available options for kubernetes:

```bash
chaos discover chaostoolkit-kubernetes --no-install
```

See `discovery.json`

### Run chaos experiment

Run chaos experiment which kills 2 pods (out of 3) and waits 90s:

```bash
chaos run --journal-path journal.json experiment.json
```

Play around, with each run `chaos run experiment.json` and see what happens:

- change replicas to 2
- in `experiment.json` change `pauses after` value to 10.

Notice to uncordon node 3 after running experiment.

### Generate report

Generate report:

```bash
chaos report --export-format=html5 journal.json report.html
```
