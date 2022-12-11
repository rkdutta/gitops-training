# gitops-training

This is an example repository for gitops implementation using Flux. This repository contains the following usecases:

1. Bootstrapping process
2. base-overlay repository example
3. kustomization example
4. helmrelease example
5. Image automation example on kustomization
6. Image automation example on helmrelease

## Bootstrapping
### Bootstrapping a flux system (default)

```
flux bootstrap github
 --owner <git-hub-user-id> 
 --repository=gitops-training 
 --personal  
 --path=clusters/cluster-0
```

## Bootstrapping a flux system (separate namespace, same cluster)
```
flux bootstrap github 
 --owner <git-hub-user-id> 
 --repository=gitops-training 
 --personal  
 --path=clusters/cluster-1
 --namespace cluster-1-flux-system
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
Reference: [kustomization example - cluster-0](clusters/cluster-0/kustomization-example.yaml)

## List all the kustomizations in the cluster
```
kubectl get kustomizations.kustomize.toolkit.fluxcd.io -A

```

## Reconcile
It is possible to reconcile a kustomization with the changes in the source repository

```
flux -n flux-system reconcile flux-system --with-source
```

## HelmRelease example
