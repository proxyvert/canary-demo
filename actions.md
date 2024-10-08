## login into okd
```
oc login -u user --server=https://api.:6443
```
## check if I have cluster-admin
```
oc auth can-i '*' '*' --all-namespaces
oc auth can-i --list
```

## install istioctl
```
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.23.2
export PATH=$PWD/bin:$PATH
```
## create kind cluster since I don't have enough privileges in okd
```
kind create cluster
kubectl cluster-info --context kind-kind


istioctl install --set profile=ambient --skip-confirmation

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.23/samples/addons/prometheus.yaml -n istio-system
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.23/samples/addons/kiali.yaml

kubectl edit configmap kiali -n istio-system
data:
  config.yaml: |
    external_services:
      prometheus:
        url: http://prometheus:9090

k rollout restart deployment kiali -n istio-system

istioctl dashboard kiali

kubectl create namespace reviews
kubectl label namespace reviews istio.io/rev=ambient --overwrite

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl create ns argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

git remote add origin https://github.com/proxyvert/canary-demo.git

kubectl apply -f argocd/applications/reviews-app.yaml

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl port-forward svc/argocd-server -n argocd 8080:443



curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x kubectl-argo-rollouts-linux-amd64
sudo mv kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts

To update the application:

1. Update the image version in `apps/reviews-app/rollout.yaml`:

yaml
spec:
  template:
    spec:
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v2:1.17.0  # Updated version


2. Commit and push the change to the Git repository.

kubectl port-forward svc/reviews 9080:9080 -n reviews

kubectl argo rollouts get rollout reviews-rollout -n reviews --watch
watch -n1 "kubectl get virtualservice reviews-vs -n reviews -o yaml | yq e '.spec.http[].route' -"
```
