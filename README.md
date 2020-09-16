# gitops-cluster1

*This serves as a proof of concept for a multi-tenant cluster managed with Git, Flux v2 and Kustomize*


The goal is to have cluster wide operations performed by the cluster administrators while the namespace scoped operations are performed by various teams each with its own Git repository. That means a team member, that's not a cluster admin, can't create namespaces, custom resources definitions or change something in another team namespace.

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

We also created two Flux custom resources, a GitRepository that fetches this repo from GitHub and a Kustomization resource that builds and applies kustomization files in the iad/nonprod directory

```
$ kubectl get gitrepositories.source.toolkit.fluxcd.io -n gitops-system
NAME            URL                                               READY     STATUS                       AGE
cluster1-repo   ssh://git@github.com/mberwanger/gitops-cluster1   Unknown   reconciliation in progress   15m

$ kubectl get kustomizations.kustomize.toolkit.fluxcd.io -n gitops-system
NAME               READY   STATUS                                                              AGE
cluster1-nonprod   True    Applied revision: master/e4105daeb4e5224101591e9dbee0f7bfb8063fe7   15m
```

In this POC, its a one to one mapping between the cluster git repository and the AWS account and it has the following structure:

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

**/base:** contains a set of resources and associated customizations that can be reused by the cluster.

**/{region}:** top level directory for a region (iad|pdx).

**/{region}/common:** contains configurations that span all clusters in the region.

**/{region}/{cluster}:** top level directory for cluster within region (nonprod|prod).

**/{region}/{cluster}/cluster:** contains configurations that are specific for this cluster in the region. We are just including the region's common configurations, but it can be customized further if needed.

**/{region}/{cluster}/{namespace}:** contains the application namespace's configuration files. Namespace-1 is using Flux to watch a team owned [git repository](https://github.com/mberwanger/gitops-team1) and provisions the resource contained within it.

**/manifest:** this directory contains the Kubernetes manifests to you use to bootstrap the cluster with Flux. If the plan is to provision the cluster via terraform this can be move there and the rest of your cluster configuration will live in this repo.

## Metrics

Prometheus is deployed as part of the cluster and metrics can be viewed through port forwarding. Use `kubectl` to map a local port from your computer that will redirect it to a specific port in your cluster.

```
kubectl port-forward -n kube-system $(kubectl get pods --selector=app=prometheus,component=server -n kube-system --output=jsonpath="{.items..metadata.name}") 9090
```

Then fire up your favorite browser and navigate to http://localhost:9090/
