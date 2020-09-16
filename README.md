# gitops-cluster1


install kind and kustomize

kind create cluster

this
manifest are used to bootstrap the cluster and point to this repo on github for

Apply CRDs
kustomize build manifest/crds | kubectl apply -f -

Bootstrap cluster
kustomize build manifest/cluster | kubectl apply -f -

gotk bootstrap github \
  --owner=$GITHUB_USER \
  --repository=gitops-cluster1 \
  --path=iad \
  --personal