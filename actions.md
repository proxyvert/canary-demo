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

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl create ns argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

git remote add origin https://github.com/proxyvert/canary-demo.git

kubectl apply -f argocd/applications/reviews-app.yaml

kubectl port-forward svc/argocd-server -n argocd 8080:443
```
