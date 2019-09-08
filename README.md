
# Distributed hackaton - chaos-toolkit

This was done during Distributed Cloud Native Hackathon 7th Sept 2019.
[Meetup event info](https://www.meetup.com/Cloud-Native-Kubernetes-Warsaw/events/264307416/)

Use case:

- get to know managed Kubernetes on Microsoft Azure
- use [chaos-toolkit](https://docs.chaostoolkit.org/) to play around

It is advised to read this document from fully before doing any cluster setups
or app deployments.

## Known limitations

- **this is a sample example not for production use, use at your own risk**.
- this was tested only on Azure, with simple k8s cluster with
  smallest instances, 1 core, 4 nodes without
- we used node draining for simulating node loss - after talking in the team
  and due to time constraints we ruled out calling to Azure API to effectively
  delete nodes
- remember to delete cluster afterwards

## Before you begin and use specific cluster

Create kubernetes cluster with 4 nodes, in Azure it should take about 30min.

Export KUBECONFIG variable so that you manage only specific cluster.

```bash
export KUBECONFIG="$(pwd)/.kube/config"
```

If using Azure and playing in shell console in the cloud just [fetch cluster credentials](https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials).

## On fresh cluster

Install helm [v2.14.3](https://github.com/helm/helm/releases/tag/v2.14.3)
on your control host.

Then we need to apply RBAC on cluster and deploy helm:

```bash
kubectl apply -f provision/kubernetes/helm-tiller.rbac.yaml

helm init --service-account tiller --wait

```

## Add azurefile storage provider

If you are using Azure k8s then we need read-write-many StorageClass to allow
multiple pods to write to storage:

```bash
kubectl apply -f provision/kubernetes/azure-pvc-roles.yaml
kubectl apply -f provision/kubernetes/azure-file-sc.yaml
```

For other storage/cloud providers you may need to adjust it accordingly.

## Deploying app

Using Bitnami Wordpress:

```bash
helm install --name wp-02 stable/wordpress -f wordpress-azurefile.yaml
```

It takes about few minutes due to the way storage is attached to k8s nodes.

## Look at the app

```bash
kubectl port-forward service/wp-02-wordpress 80:80 8081:80
kubectl get endpoints
kubectl get svc
```

Notice that Azure managed kubernetes may be exposing app publicly to the Internet.

## Apply labels on the nodes

We will use label `drain-me` as a selector when doing one of the tests later.
We just want to manage specific nodes in this scenario.

```bash
kubectl get nodes
NAME                       STATUS   ROLES   AGE   VERSION
aks-agentpool-32137755-0   Ready    agent   33m   v1.13.10
aks-agentpool-32137755-1   Ready    agent   33m   v1.13.10
aks-agentpool-32137755-2   Ready    agent   33m   v1.13.10
aks-agentpool-32137755-3   Ready    agent   33m   v1.13.10
```

See where database was deployed (kubectl describe will show it),
in our case it was node with suffix 2. So we add labels accordingly:

```bash
kubectl label nodes aks-agentpool-32137755-2 app=mysql --overwrite

kubectl label nodes aks-agentpool-32137755-0 app=drain-me --overwrite
kubectl label nodes aks-agentpool-32137755-1 app=drain-me --overwrite
kubectl label nodes aks-agentpool-32137755-3 app=drain-me --overwrite

```

## Install local dependencies for chaos toolkit

We will use [pyenv](https://github.com/pyenv/pyenv) with [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv)
to install desired python version and then to create virtualenv for the project:

```bash
pyenv virtualenv chaos
pyenv activate chaos

pip install -r requirements.txt

```

## Using chaos toolkit

### Read tutorial

When the cluster is getting ready it is good time to read about
[chaos-toolkit tutorial](https://docs.chaostoolkit.org/reference/tutorial/).

This way next sections will be less cryptic ;-)

### Run chaos discover

Remember to export `KUBECONFIG`.

First, run chaos discovery to fetch available options for kubernetes:

```bash
chaos discover chaostoolkit-kubernetes --no-install
```

See `discovery.json`

### Look at the experiment

Look into `experiment.jsont`  - we defined there:

- hypothesis that specific microservice is up and healthy, where healthy
  means it has proper number of desired replicas
- testing methods to validate above:
  - kill 2 pods and wait 90s
  - drain 1 radom node and wait 90s

### Run chaos experiment

We know that in Auzre and non given cluster the specific wordpress pods are
in Ready state in about 70s, so we will use 90s as a base time span for tests.

Run chaos experiment which kills 2 pods (out of 3) and waits 90s:

```bash
chaos run --journal-path journal.json experiment.json
```

Play around, with each run `chaos run experiment.json` and see what happens:

- change replicas to 2
- in `experiment.json` change `pauses after` value to 10.

Notice to uncordon nodes after running experiment.

Add your own experiments basing on [chaosk8s](https://docs.chaostoolkit.org/drivers/kubernetes/).

### Generate report

Generate example report:

```bash
chaos report --export-format=html journal.json report.html
```

Pro tip - there are more formats, such as: asciidoc, beamer, commonmark,
context, docbook, docx, dokuwiki, dzslides, epub, epub3, fb2, haddock,
html, html5, icml, json, latex, man, markdown, markdown_github,
markdown_mmd, markdown_phpextra, markdown_strict, mediawiki, native,
odt, opendocument, opml, org, pdf, plain, revealjs, rst, rtf, s5,
slideous, slidy, texinfo, textile
