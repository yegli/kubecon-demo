# Demo
This demo showcases how to set up and manage a Kubernetes cluster using modern tools and workflows. You'll learn how to deploy and manage applications with Headlamp and ArgoCD, and expose them to the outside world using an Ingress controller. By the end of this session, you'll have a fully functional Kubernetes environment and a clear understanding of how these tools can simplify cluster management and application deployment.

## Goal
By the end of this demo, you will:
1. Set up a Kubernetes cluster with Minikube.
2. Install and use Headlamp for cluster management.
3. Use KRO to define a Set of ResourceGraphDefinitions
4. Deploy a sample application using ArgoCD.
5. Access the application via an Ingress controller.

## Pre-Requisites
1. Install `kubectl`, `helm`, and `brew` on your machine.
2. Ensure Minikube is installed and running (`brew install minikube`).
3. Clone this repository and navigate to the directory:
   ```sh
   git clone https://github.com/yegli/kubecon-demo.git
   cd kubecon-demo
   ```

## Basics
1. install minikube: 
```
brew install minikube
```
2. run minikube cluster:
```
minikube start --driver=docker --addons=metrics-server --nodes=3
```

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
Now that your local port forwarding is set up, you can generate your RBAC token for the admin user by following [this guide](https://headlamp.dev/docs/latest/installation/#create-a-service-account-token).

Alternatively, you can integrate with an OIDC provider as documented [here](https://headlamp.dev/docs/latest/installation/in-cluster/oidc).

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


## Enabling ArgoCD Plugin
 Create a custom namespace:
 ```
 kubectl create ns argocd
 ```
 
 Then through the headlamp UI under Apps section select argo-cd and install it into your new namespace.

Retrieve the Admin User Credentials once the chart has been deployed succesfully:
```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
Setup Port Forwarding (when using both headlamp and argocd be sure to map to dedicated local ports)
```sh
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## Installing your Custom APP
Define your Application Manifests in your Repository. You can find a sample application here: https://code.swisscom.com/yanick.egli/kubecon-demo-manifests#

### Integrate with ArgoCD

#### Add Repository
Add the repository as HTTPS with the `Repository URL` set to 
```
https://github.com/yegli/kubecon-demo.git
```


Create an Application for the `argocd-root-app` subdirectory through the Web UI. This should spin up the following apps
1. ResourceGraphsDefinition (collection of all the KRO Custom API definitions)
2. Sample App 1 for Team A (default image, ingress enabled, ha enabled)
3. Sample App 2 for Team B (custom image, ingress disabled, ha disabled)

## Conclusion
Now you got to deploy your first ResourceGraphDefinition and two corresponding applications. You were able to configure both seperately but still profit from the simplicity of using the same Application Template.

## Clean Up
Once you are done exporing you can either stop your minikube cluster to resume to your development environment later by running `minikube stop`or you can completely remove the cluster by running `minikube destroy`

