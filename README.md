
# Distributed hackaton - chaos-toolkit

This was done during Distributed Cloud Native Hackathon 7th Sept 2019.
[Meetup event info](https://www.meetup.com/Cloud-Native-Kubernetes-Warsaw/events/264307416/)

It is advised to read this document fully before doing any cluster setups
or app deployments.

## Objectives

Primary:

- try to recreate demo scenario from chaos-toolkit about app on k8s with node
  pool replacement (this was on GKE, so we want to try it on Azure)

Secondary:

- get to know about managed Kubernetes on Microsoft Azure
  - prior experience almost 0
- use [chaos-toolkit](https://docs.chaostoolkit.org/) to play around,
  without prior experience with the tool

## Known limitations

- **this is a sample example not for production use, use at your own risk**.
- this was tested only on Azure, with simple k8s cluster with
  smallest instances, 1 core, 4 nodes
- we used node draining for simulating node loss - after talking in the team
  and due to time constraints we ruled out calling to Azure API to effectively
  delete nodes
- remember to delete cluster afterwards

## Main takeaways

### Objectives achievements

- original primary objective was not achieved - due to time and tech knowledge
  constraints about Azure in general
- primary objective was adjusted - we changed it from `replace node` to
  `drain selected node` - and this was acheived successfully
- secondary objective achieved, we know Azure managed k8s and
  chaos-toolkit better

### Azure

- Microsoft Azure provides managed kubernetes, but to use autoscaling node
  pools requires additional subscription features which are in Preview mode
  - we were unable to activate it, so we switched to simple node draining scenario
- Due to how DNS (sic!) was set in web spawned console in Azure
  (console icon in top right corner) we were unable to reach k8s cluster
  with kubectl, also we had no time to adjust the env which was console using
  (such as adjusting DNS and so on and walled-garden approach of that vm/
  container/whatever) due to time constrains and lack of experience
- Due to above we went full YOLO mode `¯\_(ツ)_/¯` - we decided to compress
  `~/.kube/` on the vm/container, and fetch it to the laptops -
  this is a short living cluster and it was wiped after 12h
- Using storage class `azurefile` was much faster than normal disks in Azure
- total cloud cost 4.45 EUR - 12h of compute/storage with k8s clusters,
  could be a bit less due to the fact we never touched Azure before.

### Apps and management

- Helm chart for `stable/wordpress` has very slow readiness probes

### Chaos toolkit

- REALLY READ chaos-toolkit tutorials before using it (derp mode on)
- `chaos-toolkit` looks pretty interesting in automating tests, especially
  for nightly builds
- Currently chaos-toolkit provides a lot of testing extensions especially for
  AWS, while GCP is severely lacking (actually just add/remove node pools in GKE)
- Currently chaos-toolkit looks mature in it's core but needs additional
  extendability

## Preparing infrastructure

### Creating kubernetes cluster

Create kubernetes cluster with 4 nodes, in Azure it should take about 30min.

Export KUBECONFIG variable so that you manage only specific cluster.

```bash
export KUBECONFIG="$(pwd)/.kube/config"
```

If using Azure and playing in shell console in the cloud just [fetch cluster credentials](https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials).

### Add azurefile storage provider

If you are using Azure k8s then we need read-write-many StorageClass to allow
multiple pods to write to storage:

```bash
kubectl apply -f provision/kubernetes/azure-pvc-roles.yaml
kubectl apply -f provision/kubernetes/azure-file-sc.yaml
```

For other storage/cloud providers you may need to adjust it accordingly.

### Install helm

Install helm [v2.14.3](https://github.com/helm/helm/releases/tag/v2.14.3)
on your control host.

Then we need to apply RBAC on cluster and deploy helm:

```bash
kubectl apply -f provision/kubernetes/helm-tiller.rbac.yaml

helm init --service-account tiller --wait

```

Since now your kubernetes cluster should be ready for app deployment.

## Deploying app

We will use official helm chart for wordpress with our minor customizations
within `wordpress-azurefile.yaml` file:

```bash
helm install --name wp-02 stable/wordpress -f wordpress-azurefile.yaml
```

It takes about few minutes due to the way storage is attached to k8s nodes.

### Look at the app

```bash
kubectl port-forward service/wp-02-wordpress 80:80 8081:80
kubectl get endpoints
kubectl get svc
```

Notice that Azure managed kubernetes may be exposing app publicly to the Internet.

### Apply labels on the nodes

We will use label `drain-me` as a selector when doing one of the tests later.
We just want to manage specific nodes in this scenario, and leave node with
database intact.

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

## Using chaos toolkit

### Install local dependencies for chaos toolkit

We will use [pyenv](https://github.com/pyenv/pyenv) with [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv)
to install desired python version and then to create virtualenv for the project:

```bash
pyenv virtualenv chaos
pyenv activate chaos

pip install -r requirements.txt

```

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

Look into `experiment.json`  - we defined there:

- hypothesis that specific microservice is up and healthy, where healthy
  means it has proper number of desired replicas
- testing methods to validate above:
  - kill 2 pods and wait 90s
  - drain 1 random node and wait 90s

### Run chaos experiment

We know that in Azure and non given cluster the specific wordpress pods are
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
