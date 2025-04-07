# Demo
DESCRIPTION

## Basics
1. install minikube ```brew install minikube```
2. run minikube cluster ```minikube start```
3. enable metrics server ```minikube addons enable metrics-server```

## Install Headlamp 

### InCluster Installation

```sh
helm repo add headlamp https://kubernetes-sigs.github.io/headlamp/
```
```sh
helm install headlamp headlamp/headlamp --namespace headlamp  --create-namespace -f values.yaml
```
Once installed you must setup the appropriate port forwarding to access the UI over local port 8080

```sh
export POD_NAME=$(kubectl get pods --namespace headlamp -l "app.kubernetes.io/name=headlamp,app.kubernetes.io/instance=headlamp" -o jsonpath="{.items[0].metadata.name}")
export CONTAINER_PORT=$(kubectl get pod --namespace headlamp $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
echo "Visit http://127.0.0.1:8080 to use your application"
kubectl --namespace headlamp port-forward $POD_NAME 8080:$CONTAINER_PORT
```
Now that your local port forwarding is set up you can generate your rbac token for the admin user with the following guide: https://headlamp.dev/docs/latest/installation/#create-a-service-account-token

alternatively integration with an OIDC provider is documented here: https://headlamp.dev/docs/latest/installation/in-cluster/oidc

### Local Installation
A local installation is best performed using brew for MacOS after which access to different clusters is handled based on the users permissions setout in his kubeconfig.
```sh
brew install headlamp
```

## Install KRO
The following command will install the newest version of KRO into your cluster based on your current context.
```sh
export KRO_VERSION=$(curl -sL \
    https://api.github.com/repos/kro-run/kro/releases/latest | \
    jq -r '.tag_name | ltrimstr("v")'
  )
helm install kro oci://ghcr.io/kro-run/kro/kro \
  --namespace kro \
  --create-namespace \
  --version=${KRO_VERSION}
```

## Enabling ArgoCD Plugin
 Create a custom namespace ```k create ns argocd```then through the headlamp UI under Apps section select argo-cd and install it into your new namespace.

Retrieve the Admin User Credentials:
```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
Setup Port Forwarding (when using both headlamp and argocd be sure to map to dedicated local ports)
```sh
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## Installing your Custom APP
Define your Application Manifests in your Repository and generate a project access token to allow ArgoCD to read from your Repository. You can find a sample application here: https://code.swisscom.com/yanick.egli/kubecon-demo-manifests#

### Integrate with ArgoCD
