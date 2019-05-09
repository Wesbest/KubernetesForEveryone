 Kubernetes For Everyone
-----------------------

In this workshop we will create your own namespace, deployment and service. We will also destroy a pod, get more info on services and update our pods to a newer version.

&nbsp;
###  Login to your environment


Launch the cloud shell. Use your Devoteam account to login. When the shell appear copy below, paste it there and hit enter.

Go to the following link: https://console.cloud.google.com/home/dashboard?cloudshell=true




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

You probably see something similiar above. Those are the default namespaces that are always there. If you will not specify a namespace, all your deployments and changes will be done on the default namespace. 
&nbsp;

It's now time to create your own namespace. Use the command below and change the name in the example to your own name.

```bash
kubectl create namespace wesley-rouw
```

If you have wrongly created a namespace you can remove by using the followimg command:
```
kubectl delete namepsace <namespacename>
```

Most commands are easy to use in Kubernetes, switching between namespaces is a different story. Copy the command and you will be in your namespace. Every work you will do, will now be done in this namespace. Work that will be done by others will be done in their own namespace. The command after that should confirm you are in the right namespace.

```bash
kubectl config set-context $(kubectl config current-context) --namespace=wesley-rouw
# Validate it
kubectl config view | grep namespace:
```
If you're wondering why such a long command to change to your namespace. This because Kubernetes allows you to manage multiple clusters with one account. The $(kubectl config current-context)  allows you to switch between different Kubernetes clusters.

&nbsp;
### Create a deployment
Now that we are in the right namespace we will deploy our replicaset

```bash
kubectl apply -f https://raw.githubusercontent.com/Wesbest/KubernetesForEveryone/master/Training/kubernetes_deployment.yaml
````
We have just deployed the following. It's a deployment with 3 pods. 
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubern8sdemo
  labels:
    app: kubern8sdemo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubern8sdemo
  template:
    metadata:
      labels:
        app: kubern8sdemo
    spec:
      containers:
      - name: kubern8sdemo
        image: mvdmeij/k8sdemo:v1
        ports:
        - containerPort: 80
```

To make sure if the deployment went correctly we are going to check the status

```bash
kubectl get deployments
```
If you want to delete the deployment you can use the following command: 

```bash
kubectl delete deployment <deployment_name>
```

You should see something similar like this. These are the amounts of pods you have created during this deployment. As you can see you have a desired amount of 3 pods and the current amount of pods are 3

```bash
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubern8sdemo   3         3         3            3           43s
```
&nbsp;
### Pods
Let's now take a closer look at the pods. 
```bash
kubectl get pods
```

```bash
NAME                           READY     STATUS    RESTARTS   AGE
kubern8sdemo-68fcf74b6-55vjr   1/1       Running   0          1m
kubern8sdemo-68fcf74b6-h8ncl   1/1       Running   0          1m
kubern8sdemo-68fcf74b6-qddsp   1/1       Running   0          1m
```
As you can see there are three pods created each running with an own name. What would happen if we would delete one pod? Use the command below and change the name of the pod to yours. 

```bash
kubectl delete pod kubern8sdemo-68fcf74b6-55vjr
```

Okay we have now deleted the pod. Let's take a look at the status of the pods

```bash
kubectl get pods
```

```bash
NAME                           READY     STATUS    RESTARTS   AGE
kubern8sdemo-68fcf74b6-h8ncl   1/1       Running   0          2m
kubern8sdemo-68fcf74b6-qddsp   1/1       Running   0          2m
kubern8sdemo-68fcf74b6-s4j2h   1/1       Running   0          24s
```
In my example you can see there are still three pods running, how is that possible? We have deleted a pod, but we have stated that the desired amount of the pods should be 3. Kubernetes have automatically created a new pod. Above you can see a new pod has been created 24s ago while the other 2 were created 2m ago. 
&nbsp;

So how do we reduce the amounts of pods then? We will have to scale the replicaset. We are going to scale down the deployment to 2 pods. This will be a permanent change, unless we late adjust the desired amount again.

```bash
kubectl scale deployments kubern8sdemo --replicas=2  
```
Now let's check the amount of pods 

```bash
kubectl get pods
```
```bash
NAME                           READY     STATUS    RESTARTS   AGE
kubern8sdemo-68fcf74b6-h8ncl   1/1       Running   0          7m
kubern8sdemo-68fcf74b6-qddsp   1/1       Running   0          7m
```
As you can see the amount of pods has been reduced. Let's check the deployment to see the desired amount of pods
```bash
kubectl get deployments
```
```bash
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubern8sdemo   2         2         2            2           10m
```
The desired amount is now 2. 

&nbsp;

### Create a service
We have our application ready, let's expose it to the outside world. For that we need to create a loadbalancer. This loadbalancer will be linked to the deployment with the use of the selectors and labels. Create the service:

Download the service:
```bash
wget https://raw.githubusercontent.com/Wesbest/KubernetesForEveryone/master/Training/kubernetes_service.yaml
```

First we need to manipulate the service defintion by adding an external ip adress. You can find the ip adress that is assigned to you on your sheet on your desk.

Add the external ip adress by replacing the IP_ADDRESS value by using 'sed' with the ip adresses that you have selected from the list.

```bash
sed -i 's/IP_ADDRESS/10.10.10.1/g' kubernetes_service.yaml
```

Verify if the IP_ADDRESS value has been properly replaced with the in the 'loadBalancerIP' section.

```bash
cat kubernetes_service.yaml
```
Something simimilair as the following should be your output


Let's apply the service:

```bash
kubectl create -f kubernetes_service.yaml
```


To get a clear view of what we have created I have included the yaml file of the service. 

```bash
apiVersion: v1
kind: Service
metadata:
  name: kubern8sservice
  labels:
    app: kubern8sdemo
spec:
  ports:  
  - port: 80
    targetPort: 80
  selector:
    app: kubern8sdemo
  type: LoadBalancer
   LoadBalancerIP: 35.242.185.86 
```
To check the status of the service, use the command below.

```bash
kubectl get service
```

```bash
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
kubern8sservice   LoadBalancer   10.51.240.199   35.242.185.86   80:32015/TCP   47s
```
Something similar should appear. The loadbalancer is now created and has got an external ip. If it says pending, just wait a little longer it should appear soon.
&nbsp;

If you entered the wrong ip adress in your service manifest then you can delete the service by using the following command:

```bash
kubectl delete service kubern8sservice
```

We have connected the service to the pods with the labels and selectors. When we will check our browser we will see the application. Open a new tab and copy the external IP. A webapplication should appear. Hit the autorefresh button. We will need that later. 

&nbsp;

### Update Pods

Don't you think our application is outdated? I've heard the image got some updates, let's check it out. Remember the deployment?

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubern8sdemo
  labels:
    app: kubern8sdemo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubern8sdemo
  template:
    metadata:
      labels:
        app: kubern8sdemo
    spec:
      containers:
      - name: kubern8sdemo
        image: mvdmeij/k8sdemo:v1
        ports:
        - containerPort: 80
```

As you can see it has version 1. Currently they are already on version 5. With a single command we can update the the pods. When you do so, don't forget to check your browser. Before proceeding to the next steps, wait till the application has been updated in your browser.

```bash
kubectl set image deployment/kubern8sdemo kubern8sdemo=mvdmeij/k8sdemo:v5
```
I think version 5 is a little bit too overwhelming, we will switch to an older version. When we have done so, we will check out the status of the pods to see what happens under the hood when we do a rollout of an older or newer version to the pods.  
```bash
kubectl set image deployment/kubern8sdemo kubern8sdemo=mvdmeij/k8sdemo:v3
```
```bash
kubectl get pods -w
```
the -w means that we are watching for changes. when you have executed that command you should see something similar like this:
```bash
NAME                           READY     STATUS              RESTARTS   AGE
kubern8sdemo-7b4db456b-j59sd   1/1       Running             0          2m
kubern8sdemo-7b4db456b-npk75   1/1       Running             0          2m
kubern8sdemo-84ff7fd64-w4ksc   0/1       ContainerCreating   0          2s
kubern8sdemo-84ff7fd64-w4ksc   1/1       Running             0          3s
kubern8sdemo-7b4db456b-j59sd   1/1       Terminating         0          2m
kubern8sdemo-84ff7fd64-m4rzh   0/1       Pending             0          0s
kubern8sdemo-84ff7fd64-m4rzh   0/1       Pending             0          0s
kubern8sdemo-84ff7fd64-m4rzh   0/1       ContainerCreating   0          0s
kubern8sdemo-7b4db456b-j59sd   0/1       Terminating         0          2m
kubern8sdemo-7b4db456b-j59sd   0/1       Terminating         0          2m
kubern8sdemo-7b4db456b-j59sd   0/1       Terminating         0          2m
kubern8sdemo-84ff7fd64-m4rzh   1/1       Running             0          4s
kubern8sdemo-7b4db456b-npk75   1/1       Terminating         0          2m
kubern8sdemo-7b4db456b-npk75   0/1       Terminating         0          2m
kubern8sdemo-7b4db456b-npk75   0/1       Terminating         0          2m
kubern8sdemo-7b4db456b-npk75   0/1       Terminating         0          2m
```
Kubernetes is creating a new container and then terminates one and does this one more time since our desired amount is 2 containers. Use control/cmd c to stop it. 

To review the metrics of the pods in your namespace you can use the following command. (It can take a small amount of time before metrics are shown)

```bash
kubectl top pod
```

#### Clean up your mess

Try to delete your deployment(s) and service(s) by using the cheat sheet. 



```do it yourself```

Try to get Mario running in your browser, or checkout https://www.dockerhub.com for your own preferred application which you want to run in an container.

Requirements: 
Create a deployment with 1 container.
Create a services with type  the LoadBalancer and re-use your public ip adress. 

Image source: Nhttps://hub.docker.com/r/pengbai/docker-supermario/ 

Don't reinvent the wheel. Use our templates. ->> MOET BITLY WORDEN (https://github.com/Wesbest/KubernetesForEveryone/tree/master/Templates)

Use Nano as an editor in Cloudshell.. for the CommandF00  masters among us use vim!




&nbsp;
### The End
Well that's about it. You have learnt how to set up your own namespace. In this namespace you have set up a pod with 3 containers. Later we have scaled this amount down to 2. After that you have set up your own loadbalancer. Kubernetes automatically detected to which pod it needs to be connected. At last we have updated the version of the images the containers were running. In case you have questions, please feel free to ask.