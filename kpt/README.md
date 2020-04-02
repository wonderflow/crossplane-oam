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

### Sync with Remote App Repository

When some changes occurred both local and remote apps, you could sync and merge with KPT.

https://googlecontainertools.github.io/kpt/guides/consumer/update/#commit-local-changes



