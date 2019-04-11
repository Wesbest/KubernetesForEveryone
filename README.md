 Kubernetes For Everyone
-----------------------

In this workshop we will create your own namespace, deployment and service. We will also destroy a pod, get more info on services and update our pods to a newer version.


###  Login to your environment:
Goto this link: https://cloud.google.com/shell/

Launch the cloud shell. Use your Devoteam account to login. When the shell appear copy 
```bash
gcloud container clusters get-credentials standard-cluster-1 --zone us-central1-a --project valid-delight-234309
```
paste it there and hit enter.

### Create a namespace
Now that we are into the cluster we need to create a namespace, but first let's see what namespaces are there already.

```bash
kubectl get namespace
```
```bash
NAME          STATUS    AGE
default       Active    1d
kube-system   Active    1d
kube-public   Active    1d
```
You probably see something similiar above. It's now time to create your own namespace. Use the command below and change the name in the example to your own name.

```bash
kubectl create namespace <wesley-rouw>
```
Most commands are easy to use in Kubernetes, switching between namespaces is quite difficult. Copy the command and you will be in your namespace. The command after that should confirm you are in the right namespace.

```bash
kubectl config set-context $(kubectl config current-context) --namespace=wesley-rouw
# Validate it
kubectl config view | grep namespace:
```

### Create a deployment
Now that we are in the right namespace it is time that we will deploy 
