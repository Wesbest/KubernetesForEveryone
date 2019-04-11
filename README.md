 Kubernetes For Everyone
-----------------------

In this workshop we will create your own namespace, deployment and service. We will also destroy a pod, get more info on services and update our pods to a newer version.

&nbsp;
###  Login to your environment
Goto this link: https://cloud.google.com/shell/

Launch the cloud shell. Use your Devoteam account to login. When the shell appear copy below, paste it there and hit enter.
```bash
gcloud container clusters get-credentials kubeforeveryone --zone europe-west2-a --project dulcet-provider-225307
```

&nbsp;
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

&nbsp;
### Create a deployment
Now that we are in the right namespace it is time that we will deploy our replicaset

```bash
kubectl apply -f https://raw.githubusercontent.com/Wesbest/KubernetesForEveryone/master/Training/kubernetes_deployment1.yaml
````
We have just deployed the following. It's a deployment with 3 pods. 
```bash
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hello-node
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: hello-node
    spec:
      containers:
      - name: hello-node
        image: mvdmeij/k8sdemo:1
        ports:
        - containerPort: 8080
```

To make sure if the deployment went correctly we are going to check the status

```bash
kubectl get deployments
```
You should see something similar like this. These are the amounts of pods you have created during this deployment. As you can see you have a desired amount of 3 pods and the current amount of pods are 3

```bash
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   3         3         3            3           6s
```
&nbsp;
### Pods
Let's now take a closer look at the pods. 
```bash
kubectl get pods
```

```bash
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-7ccb79b494-95qmc   1/1       Running   0          12s
hello-node-7ccb79b494-ht8vz   1/1       Running   0          12s
hello-node-7ccb79b494-z7sz9   1/1       Running   0          12s
```
As you can see there are three pods created each running with an own name. What would happen if we would delete one pod? Change the name to one of your pods

```bash
kubectl delete pod <hello-node-7ccb79b494-95qmc>
```

Okay we have now deleted the pod. Let's take a look at the status of the pods

```bash
kubectl get pods
```

```bash
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-7ccb79b494-ht8vz   1/1       Running   0          13m
hello-node-7ccb79b494-shv8q   1/1       Running   0          49s
hello-node-7ccb79b494-z7sz9   1/1       Running   0          13m
```
In my example you can see there are still three pods running, how is that possible? We have deleted a pod, but we have stated that the desired amount of pods should be 3. Kubernetes have automatically create a new pod. Above you can see a new pod has been created 49s ago while the other 2 were created 13m ago. 
