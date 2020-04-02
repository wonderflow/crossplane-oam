# Using OAM by KPT

Using KPT to play with OAM Apps.

## Install OAM Crossplane runtime

You could easily install OAM Crossplane runtime by following our [installation guides](../README.md#Installation).

## Using OAM App Repository by KPT

### Fetch OAM app from remote Repository

Assuming you have already built an OAM App repository by GitHub.

Using our [example repository](https://github.com/wonderflow/crossplane-oam/tree/master/kpt/repository/) for this demo.

You could fetch OAM app from remote Repository using [kpt pkg get](https://googlecontainertools.github.io/kpt//reference/pkg/get).

#### Command

```
kpt pkg get https://github.com/wonderflow/crossplane-oam/kpt/repository/sampleapp sampleapp
```

#### Output

```
fetching package kpt/repository/sampleapp from https://github.com/wonderflow/crossplane-oam to sampleapp
```

```
➜  kpt tree sampleapp
sampleapp
├── Kptfile
├── appconfig.yaml
├── component.yaml
├── traitdef.yaml
└── workloaddef.yaml

0 directories, 5 files
```

### Install sample app

```
$ kubectl apply -f sampleapp/
applicationconfiguration.core.oam.dev/example-appconfig created
component.core.oam.dev/example-component created
traitdefinition.core.oam.dev/manualscalertraits.core.oam.dev created
workloaddefinition.core.oam.dev/containerizedworkloads.core.oam.dev created
```

You will get a K8s deployment as real running resource which produced by OAM Crossplane runtime.

```
$ kubectl get deploy
NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
example-appconfig-workload-deployment   3/3     3            3           114s
```


### Sync with Remote App Repository

When some changes occurred both local and remote apps, you could sync and merge with KPT.

For example, we changed our sampleapp and tag it as `v0.1.0`. Then our local sampleapp also be changed.



```
kpt pkg update sampleapp@v0.1.0 --strategy=resource-merge
```

Ref to [update section](https://googlecontainertools.github.io/kpt/guides/consumer/update/#commit-local-changes) of KPT for more details.



### Parameter Setting

[KPT Setters](https://googlecontainertools.github.io/kpt/guides/consumer/set/) also have magic power along with OAM parameters.

OAM parameter is very important feature as it allows App Developer could tell their demand to App Operator.

For example, in our sampleapp, App Developer create a component like below:

```
---
apiVersion: core.oam.dev/v1alpha2
kind: Component
metadata:
  name: example-component
spec:
  workload:
    apiVersion: core.oam.dev/v1alpha2
    kind: ContainerizedWorkload
    spec:
      containers:
        - name: my-nginx
          image: nginx:1.16.1
          resources:
            limits:
              memory: "200Mi"
          ports:
            - containerPort: 4848
              protocol: "TCP"
          env:
            - name: WORDPRESS_DB_PASSWORD
              value: ""
  parameters:
    - name: instance-name
      required: true
      fieldPaths:
        - metadata.name
    - name: image
      fieldPaths:
        - spec.containers[0].image
```

The App Developer left two parameters which are `instance-name` and `image`.

Then in AppConfig, App Operator could overwrite this parameter like below.

```
---
apiVersion: core.oam.dev/v1alpha2
kind: ApplicationConfiguration
metadata:
  name: example-appconfig
spec:
  components:
    - componentName: example-component
      parameterValues:
        - name: instance-name
          value: example-appconfig-workload
        - name: image
          value: nginx:latest
      traits:
        - trait:
            apiVersion: core.oam.dev/v1alpha2
            kind: ManualScalerTrait
            metadata:
              name:  example-appconfig-trait
            spec:
              replicaCount: 3
```

With KPT setters, this feature become more clear and useful. And you don't need to declare parameters in OAM yamls. Let's go through with it.

#### Create setter by App Developer

As the app developer need to add two parameter for his component, we add two KPT setters here:

```
$ kpt cfg create-setter sampleapp/ instance-name example-component --field "metadata.name" --description "use to set an instance name" --set-by "sampleapp developer"
```

```
$ kpt cfg create-setter sampleapp/ image nginx:1.16.1 --field "spec.workload.spec.containers[0].image" --description "use to set image for component" --set-by "sampleapp developer"
```

Then the app operator could see which parameters are available in this component like below:

```
$ kpt cfg list-setters sampleapp/
  NAME              VALUE               SET BY                   DESCRIPTION             COUNT
  image           nginx:1.16.1        sampleapp developer   use to set image for component   0
  instance-name   example-component   sampleapp developer   use to set an instance name      1
```

It's very clear and easy to understand.

#### Set Value by App Operator

Then the App Operator could set `instance-name` with a new name like this:

```
$ kpt cfg set sampleapp/ instance-name test-component
set 1 fields
```

Check the component and you will find the instane name has been changed.

```
$ cat sampleapp/component.yaml
apiVersion: core.oam.dev/v1alpha2
kind: Component
metadata:
  name: test-component # {"$ref":"#/definitions/io.k8s.cli.setters.instance-name"}
spec:
  parameters:
  - name: instance-name
    fieldPaths:
    - metadata.name
...
```

Currently array fieldpath seems don't work and we have created an [issue](https://github.com/GoogleContainerTools/kpt/issues/469).


### App Overview

With kpt, you could see an overview of your App.

```
$ kpt cfg count sampleapp
ApplicationConfiguration: 1
Component: 1
TraitDefinition: 1
WorkloadDefinition: 1
```

So in the sampleapp, we have one ApplicationConfiguration, Component, TraitDefinition and WorkloadDefinition.


### Live apply

kpt includes the next-generation **apply** commands developed out of the Kubernetes [cli-utils](https://github.com/kubernetes-sigs/cli-utils) repository as the [`kpt live apply`](https://googlecontainertools.github.io/kpt/reference/live/apply) command.

This means with `kpt live apply` command, we could wait for the controller reconcile.

```
$ kpt live init sampleapp
Initialized: ../sampleapp/grouping-object-template.yaml
```

```
$ kpt live apply sampleapp --wait-for-reconcile
configmap/grouping-object-a4e18a93 created
applicationconfiguration.core.oam.dev/example-appconfig configured
component.core.oam.dev/test-component created
traitdefinition.core.oam.dev/manualscalertraits.core.oam.dev unchanged
workloaddefinition.core.oam.dev/containerizedworkloads.core.oam.dev unchanged
5 resource(s) applied. 2 created, 2 unchanged, 1 configured
configmap/grouping-object-a4e18a93 is Current: Resource is always ready
applicationconfiguration.core.oam.dev/example-appconfig is Current: Resource is current
```

### Release your OAM apps

1. Create a github repo.

2. Commit and push your sample app.

```
git add sampleapp/
git commit -m "my sampleapp commit"
git remote add origin git@github.com:<your-account>/<your-app-repo>.git
git push -u origin master
```