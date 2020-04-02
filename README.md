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

## Try Sample OAM AppConfig

```
$ kubectl apply -f samples/sample_application_config.yaml
traitdefinition.core.oam.dev/manualscalertraits.core.oam.dev created
workloaddefinition.core.oam.dev/containerizedworkloads.core.oam.dev created
component.core.oam.dev/example-component created
applicationconfiguration.core.oam.dev/example-appconfig created
```

You will get containerized workload and manual scaler trait.

```
$ kubectl get manualscalertraits.core.oam.dev
NAME                      AGE
example-appconfig-trait   4s
```

```
$ kubectl get containerizedworkloads.core.oam.dev
NAME                         AGE
example-appconfig-workload   58s
```

And you will get a K8s deployment as real running resource.

```
$ kubectl get deploy
NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
example-appconfig-workload-deployment   3/3     3            3           114s
```

# Using OAM with KPT

You could find lots of interesting things could be done by [using OAM with KPT](kpt/README.md).