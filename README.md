# gitops-cluster1

*This serves as a proof of concept for amulti-tenant cluster managed with Git, Flux v2 and Kustomize*


Cluster wide operations are performed by the cluster administrators while the namespace scoped operations are performed by various teams each with its own Git repository. That means a team member, that's not a cluster admin, can't create namespaces, custom resources definitions or change something in another team namespace.

## Installation

First apply the Flux custom resource definitions with kubectl kustomize:

`kubectl apply -k manifest/crds`

And then install the cluster wide Flux configuration:

`kubectl apply -k manifest/cluster`

That's it, you are all done.

*Please note that it may take a couple of minutes for all the components to download and startup*

## Tour de Cluster

Flux related resources are placed in the gitops-system namespace:

```
$ kubectl get all -n gitops-system
NAME                                           READY   STATUS    RESTARTS   AGE
pod/helm-controller-7dbdfb6b66-msvdz           1/1     Running   0          24m
pod/kustomize-controller-7b558f7cbd-27jz4      1/1     Running   0          24m
pod/notification-controller-77fdbd5c4b-4h2n5   1/1     Running   0          24m
pod/source-controller-758cc7d4cf-pwdmw         1/1     Running   0          24m

NAME                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/notification-controller   ClusterIP   10.109.203.213   <none>        80/TCP    25m
service/source-controller         ClusterIP   10.98.91.100     <none>        80/TCP    25m
service/webhook-receiver          ClusterIP   10.99.135.35     <none>        80/TCP    25m

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/helm-controller           1/1     1            1           25m
deployment.apps/kustomize-controller      1/1     1            1           25m
deployment.apps/notification-controller   1/1     1            1           25m
deployment.apps/source-controller         1/1     1            1           25m

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/helm-controller-7dbdfb6b66           1         1         1       24m
replicaset.apps/kustomize-controller-7b558f7cbd      1         1         1       24m
replicaset.apps/notification-controller-77fdbd5c4b   1         1         1       24m
replicaset.apps/source-controller-758cc7d4cf         1         1         1       24m
```

We also created two Flux custom resources, a GitRepository that fetches this repo from GitHub and a Kustomization resource that builds, and applies kustomization files in the iad/nonprod directory

```
$ kubectl get gitrepositories.source.toolkit.fluxcd.io -n gitops-system
NAME            URL                                               READY     STATUS                       AGE
cluster1-repo   ssh://git@github.com/mberwanger/gitops-cluster1   Unknown   reconciliation in progress   15m

$ kubectl get kustomizations.kustomize.toolkit.fluxcd.io -n gitops-system
NAME               READY   STATUS                                                              AGE
cluster1-nonprod   True    Applied revision: master/e4105daeb4e5224101591e9dbee0f7bfb8063fe7   15m
```

Repository structure:

```
├── base
│   ├── ...
├── iad
│   ├── common
│   │   └── cluster
│   │       └── kustomization.yaml
│   ├── nonprod
│   │   ├── cluster
│   │   │   └── kustomization.yaml
│   │   ├── namespace-1
│   │   │   ├── flux.yaml
│   │   │   ├── kustomization.yaml
│   │   │   └── namespace.yaml
│   │   └── namespace-2
│   │       └── ...
│   └── prod
│       └── ...
├── manifest
│   ├── ...
└── pdx
    └── ...
```

base

region

cluster

common

manifest