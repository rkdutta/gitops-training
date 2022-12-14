# gitops-training

This is an example repository for gitops implementation using Flux. This repository contains the following usecases:

1. Bootstrapping process
2. base-overlay repository example
3. kustomization example
4. helmrelease example
5. Image automation example on kustomization
6. Image automation example on helmrelease


## Install flux CLI
MacOS​
```
brew install fluxcd/tap/flux​
```
Linux​
```
curl -s https://fluxcd.io/install.sh | sudo bash​
```
Windows​
```
choco install flux
```


## Bootstrapping flux-system

### Exporting GitHub personal access token
export GITHUB_TOKEN=<your-token>

### Bootstrapping a flux system (default)

```
flux bootstrap github \
 --components-extra=image-reflector-controller,image-automation-controller \
 --owner <git-hub-user-id> \
 --repository=gitops-training \
 --personal  \
 --path=clusters/dev
```

## Bootstrapping a flux system (separate namespace, same cluster)
```
flux bootstrap github \
 --components-extra=image-reflector-controller,image-automation-controller \
 --owner <git-hub-user-id> \ 
 --repository=gitops-training \
 --personal  \
 --path=clusters/dev \
 --namespace dev-flux-system 
```

## Create namespaces
First it is good to create the namespaces
```
kubectl apply -f namespace.yaml
```

## Base and Overlays 
## Defining an application base & overlays for deployment 

The application base is defined in the following base path and a cluster deployment in the same git repository
```
├── apps
│   └── sample
│       ├── kustomization.yaml
│       ├── sample-namespace.yaml
│       └── sample-pod.yaml
├── clusters
│   ├── cluster-0
│   │   ├── flux-system
│   │   │   ├── gotk-components.yaml
│   │   │   ├── gotk-sync.yaml
│   │   │   └── kustomization.yaml
│   │   └── kustomization-example.yaml
```

## Kustomization Example
## Create a kustomization
When the flux-systems are up and running and base application is defined, it is time to deploy an application on the cluster using flux. 

Example: 
```
apiVersion: kustomize.toolkit.fluxcd.io/v1beta1
kind: Kustomization
metadata:
  name: sample-ks
  namespace: flux-system
spec:
  interval: 5m0s
  path: ./apps/sample
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system 
  validation: client
  patchesStrategicMerge:
    - apiVersion: v1
      kind: Pod
      metadata:
        name: sample-pod
        namespace: sample
        annotations:
          cluster-name: cluster-0
```
Reference: [kustomization example](clusters/dev/kustomization-sample-pod-example.yaml)

## List all the kustomizations in the cluster
```
kubectl get kustomizations.kustomize.toolkit.fluxcd.io -A

```

## Reconcile
It is possible to reconcile a kustomization with the changes in the source repository

```
flux -n flux-system reconcile ks flux-system --with-source
flux -n <kustomization namespace> reconcile ks <kustomization name> --with-source
```

## HelmRelease example
### Create repositories
Put a [repositories kustomization](clusters/dev/kustomization-repositories.yaml) in the clusters/dev folder that will create a kustomuzation to create all the source resources in flux. The definitons for the repositories are located in the [repositories](./repositories/) folder. It is important to note that the repositories are created in the management namespace

### Create a HelmRelease
Put the HelmRelease in the clusters/dev folder. flux-system in the cluster is going to deploy the HelmRelease in the cluster.

Reference: Reference: [HelmRelease example](clusters/dev/helm-release-example.yaml)


Next, force the reconciliation process.
```
flux -n flux-system reconcile ks flux-system --with-source
```

List down all helm repositories in the cluster. A helm repository representing podinfo helm repo should exist.
```
flux get sources helm -A
```

List up all helm releases
```
helm ls -A

or,

flux get helmreleases -A

or,

kubectl get helmreleases.helm.toolkit.fluxcd.io -A

```

## Image Automation
There are three things to follow:

1. Define the [image repository](./repositories/image-repository.yaml). This is created as a [kustomization](./clusters/dev/kustomization-repositories.yaml) for repository.
2. Create [image update policy](./image-update-automation/image-policy.yaml). This is created as a [kustomization](./clusters/dev/kustomization-image-automation.yaml)
3. Create the [automation operator](./image-update-automation/helm-image-update-automation.yaml). This is created as a [kustomization](./clusters/dev/kustomization-image-automation.yaml)
4. Link deployments with policy using [policy comment marker][reference](./clusters/dev/helm-release-example.yaml)
```yaml
      version: "6.2.3" # {"$imagepolicy": "management:podinfo:tag"}
```



