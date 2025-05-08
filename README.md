#



## ArgoCD

[Argo CD](https://argo-cd.readthedocs.io/en/stable/) is a declarative, GitOps-based continuous delivery tool for Kubernetes that synchronizes application state.

## setup argocd

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.14.5/manifests/install.yaml
```

## Install TEG
```sh
cat <<EOF | kubectl apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: tetrate-envoy-gateway
  namespace: argocd
spec:
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:     
      - CreateNamespace=true
      - ServerSideApply=true
  source:
    chart: teg-envoy-gateway-helm
    repoURL: docker.io/tetrate
    targetRevision: v0.0.0-latest
    helm:
      releaseName: teg
  destination:
    server: "https://kubernetes.default.svc"
    namespace: envoy-gateway-system
EOF
```


## Cert-Manager

```sh
VERSION=v1.17.0

helm repo add jetstack https://charts.jetstack.io --force-update
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version ${VERSION} \
  --set crds.enabled=true \
  --set "extraArgs={--enable-gateway-api}"
```