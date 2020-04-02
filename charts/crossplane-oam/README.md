# Install Crossplane and OAM

Crossplane can be easily installed into any existing Kubernetes cluster using
the regularly published Helm chart. The Helm chart contains all the custom
resources and controllers needed to deploy and configure Crossplane.

Crossplane and OAM all in one.

## Pre-requisites

* [Kubernetes cluster](https://kubernetes.io/docs/setup/)
  * For example
    [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/),
    minimum version `v0.28+`
* [Helm 3](https://helm.sh/docs/intro/), minimum version `v3.0.0+`.


## Installation

```console
helm repo add oam https://wonderflow.github.io/crossplane-oam/archives/
kubectl create namespace oam-system
helm install crossplane --namespace oam-system oam/crossplane-oam
```
