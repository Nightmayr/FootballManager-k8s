# Deploying to Kubernetes

We will look at how to get the kubernetes manifest files in this repository deployed using a local cluster, as well as deploying to Google Kubernetes Engine (GKE).


## Minikube

Download and install minikube. This will differ depending on your operating system.

### macOS

Install the homebrew package manager
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
Install kubectl
```
brew install kubectl
```
Download and install Virtualbox, we can use brew to accomplish this or download it directly from oracle.
```
brew install cask
brew cask install virtualbox
```
Install minikube
```
brew cask install minikube
```
Start minikube
```
minikube start
```

### Linux
Open a terminal and run the following commands
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
  ```
Add minikube to PATH
```
sudo cp minikube /usr/local/bin && rm minikube
```

### Windows
We will use the chocolatey package manger to install minikube

In Powershell (run as Administrator), run `Get-ExecutionPolicy`. If it returns `Restricted`, then run the following command

```
Set-ExecutionPolicy AllSigned
```
Or
```
Set-ExecutionPolicy Bypass -Scope Process
```

We are now able to install the chocolatey manger
```
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

Use chocolatey to install minikube and kubectl
```
choco install minikube kubernetes-cli
```
After Minikube has finished installing, close the current CLI session and restart. Minikube should have been added to your path automatically

### Deploying to minikube

Now that we have successfully installed minikube on our machines, we have a local kubernetes cluster that we can use to deploy our containers to.

begin by starting minikube

```
minikube start
```

Check that cluster is up using `kubectl`

```
kubectl get nodes
```
We should see one node.

Use git to clone this repo to your machine. Navigate inside of the cloned repo. We will apply the kubernetes manifests to our cluster.

```
kubectl apply -f .
```

Check whether the objects are being created from the maifest files

```
kubectl get all
```
once the state of all the objects is stated as being ready, we should be able to see our application running from within a browser. The IP required to access the application is the IP of our minikube cluster

```
minikube ip
```

Enter the IP into your browser. 

Congratulations you have successfully deployed the application to your local kubernetes cluster. We will now transition to getting the application hosted on the cloud using GKE.


## GKE

Create a cluster using the GKE console. Once the cluster has been created open up a cloud-shell instance and check the cluster is up and running.

```
kubectl get nodes
```

The default setup provides 3 nodes in GKE.

Clone this repo to your cloud-shell instance and navigate to the repo. Once inside apply the manifest files using `kubectl`

```
kubectl apply -f .
```

To access our application uisng a public IP we will require nginx-ingress which we will obtain using helm.

### Install Helm

```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```

### Tiller service account

```
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

helm init --service-account tiller --upgrade
```
### Install nginx-ingress using helm

```
helm install stable/nginx-ingress --name my-nginx --set rbac.create=true
```

```
POD_NAME=$(kubectl get pods -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD_NAME -- /nginx-ingress-controller --version
```

The nginx-ingresss should have provisioned a load-balancer for us. We will check our services to see whether this the case, and to get access to the public IP, which we will use to access our application.

## CI/CD Pipeline using Jenkins and Kubenetes

WORK IN PROGRESS
