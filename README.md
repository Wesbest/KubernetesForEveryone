 Kubernetes For Everyone
-----------------------
In this workshop you will create your own deployment. It will contain several pods that will run an application. The application will look like this:
&nbsp;

![Application](https://github.com/Wesbest/KubernetesForEveryone/blob/master/Pictures/App_1_star.png)

&nbsp;

In this workshop we will make use of the Google Cloud Shell. This shell provides all the tools that are necessary to manage Kubernetes. In the shell we will use the utility kubectl. On the cheat sheet you will find the useful commands that you can use during the workshop. 

&nbsp;
###  Login to your environment
Open an icognito browser, paste the following link and sign in with the account that can be find on your desk https://console.cloud.google.com/home/dashboard?cloudshell=true


```bash
gcloud beta container clusters get-credentials k8s4you --region europe-west4 --project directed-truck-196609
```
![Cloud Shell](https://github.com/Wesbest/KubernetesForEveryone/blob/master/Pictures/CloudShell.png)

&nbsp;

In case you get into the console and not into the shell. Please click on the left icon from the range below. 

![Cloud Shell](https://github.com/Wesbest/KubernetesForEveryone/blob/master/Pictures/CloudShell2.png)



&nbsp;
### Create a namespace
Now that we are into the cluster we need to create a namespace, but first let's see what namespaces are available.

```bash
kubectl get namespace
```
```bash
NAME          STATUS    AGE
default       Active    1d
kube-system   Active    1d
kube-public   Active    1d
```

You probably see something similar above. Those are the default namespaces that are always there. By default if you don't specify a namespace, all your deployments and changes will be done in the default namespace. 
&nbsp;

Now lets create your own namespace. Use the command below and change the name in the example to your own name.

```bash
kubectl create namespace max-vd-meij
```

Most commands are easy to use in Kubernetes, switching between namespaces is a different story. Copy and execute the command below to switch to your own namespace. Don't forget to switch the namespace name to your own.
The following kubectl commands will now only executed within this namespace. Deployments in other namespaces will not be visible. Validate the namespace change with the second command.

```bash
kubectl config set-context $(kubectl config current-context) --namespace=max-vd-meij
# Validate it
kubectl config view | grep namespace:
```
 
&nbsp;
### Create a deployment
Now that we are in the right namespace we will deploy our replicaset. A replicaset is part of the manifest that tells kubernetes how many copies of your pods should always be available.

```bash
kubectl apply -f https://raw.githubusercontent.com/Wesbest/KubernetesForEveryone/master/Training/k8s_deployment.yaml
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

To make sure the deployment went correctly we are going to check the status:

```bash
kubectl get deployments
```
If you want to delete the deployment you can use the following command: 

```bash
kubectl delete deployment <deployment_name>
```

If you didn't delete the deployment the status should look something similar to this. These are the amounts of pods you have created during this deployment. As you can see you have a desired amount of 3 pods and the current amount of pods are 3. 

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
As you can see there are three pods created each running with their own name. What would happen if we would delete one pod? Use the command below and change the name of the pod to yours. 

```bash
kubectl delete pod kubern8sdemo-68fcf74b6-55vjr
```

Okay we have now deleted the pod. Let's take a look at the status of the pods.

```bash
kubectl get pods
```

```bash
NAME                           READY     STATUS    RESTARTS   AGE
kubern8sdemo-68fcf74b6-h8ncl   1/1       Running   0          2m
kubern8sdemo-68fcf74b6-qddsp   1/1       Running   0          2m
kubern8sdemo-68fcf74b6-s4j2h   1/1       Running   0          24s
```
In this example you can see there are still 3 pods running, how is that possible? We have deleted a pod, but we have stated that the desired amount of the pods should be 3. Kubernetes has automatically created a new pod. Above you can see a new pod has been created 24s ago while the other 2 were created 2m ago. The kubernetes master node will make sure to keep as many available as is desired.
&nbsp;

So how do we reduce the amounts of pods? We will have to scale the replicaset. We are going to scale down the deployment to 2 pods. This will be a permanent change, unless we later adjust the desired amount of pods again.

```bash
kubectl scale deployments kubern8sdemo --replicas=2  
```
Now let's check the amount of pods again

```bash
kubectl get pods
```
```bash
NAME                           READY     STATUS    RESTARTS   AGE
kubern8sdemo-68fcf74b6-h8ncl   1/1       Running   0          7m
kubern8sdemo-68fcf74b6-qddsp   1/1       Running   0          7m
```
As you can see the amount of pods has been reduced. Let's check the deployment to see the desired amount of pods.
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
We have our application ready, let's expose it to the outside world. In order to do that we need to create a loadbalancer. This loadbalancer will be linked to the deployment with the use of the selectors and labels. Create the service:

Download the service:
```bash
wget https://raw.githubusercontent.com/Wesbest/KubernetesForEveryone/master/Training/k8s_service.yaml
```

First we need to manipulate the service definition by adding an external ip adress. You can find the ip adress that is assigned to you on your sheet on your desk. 

Add the external ip adress by replacing the IP_ADDRESS value by using 'sed' with the ip adresses that you have selected from the list.

```bash
sed -i 's/IP_ADDRESS/10.10.10.1/g' k8s_service.yaml
```

Verify the IP_ADDRESS value has been replaced properly with the'loadBalancerIP' section.

```bash
cat k8s_service.yaml
```
The final service description should look something to this:

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
  loadBalancerIP: 10.10.10.1
```
Let's apply the service:

```bash
kubectl create -f k8s_service.yaml
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

We have now connected the service to the pods with the help of labels and selectors. We can now access our application from the browser as it is exposed through the service loadbalancer. Open a new tab and copy the external IP. A webapplication should appear. Enable the autorefresh button. We will need that later. 
&nbsp;

![Application](https://github.com/Wesbest/KubernetesForEveryone/blob/master/Pictures/App_1_star.png)

&nbsp;

### Update Pods

Our developers have worked hard to create a new version of our application. Let's update the deployment! Remember the deployment?

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

As you can see our old deployment uses docker image with tag v1. The developers are already finished with version 5. With a single command we can update the version of our deployed pods. If you want to see how quick the application is updated, make sure to keep an eye on your browser. Before proceeding to the next step, wait till the application has been updated in your browser.

```bash
kubectl set image deployment/kubern8sdemo kubern8sdemo=mvdmeij/k8sdemo:v5
```
![Application](https://github.com/Wesbest/KubernetesForEveryone/blob/master/Pictures/App_5_stars.png)

&nbsp;

It seems that version 5 was a bit too much, lets roll back a few version and see what kubernetes does under the hood. If you want to watch live changes to the pods include -w as shown below. Keep an eye on the terminal while executing the version change.

```bash
kubectl get pods -w
```
```bash
kubectl set image deployment/kubern8sdemo kubern8sdemo=mvdmeij/k8sdemo:v3
```

While watching the live changes, you should see something similar to this:
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
Kubernetes is creating a new container before terminating the old one until our desired amount is reached. Use control/cmd c to stop it the pod watch.
&nbsp;

Below you can see how your application should now look in the browser:

![Application](https://github.com/Wesbest/KubernetesForEveryone/blob/master/Pictures/App_3_stars.png)

&nbsp;

### Clean up!
Clean up your deployment and your service before you go to the next task. 

&nbsp;

### Do It Yourself

![Super Mario](https://github.com/Wesbest/KubernetesForEveryone/blob/master/Pictures/SuperMario.png)

Try to get Super Mario running in your browser.

*tips

Image source: https://hub.docker.com/r/pengbai/docker-supermario/ 
Don't reinvent the wheel. Use our templates. https://github.com/Wesbest/KubernetesForEveryone/tree/master/Templates
Use Nano as text editor in Cloudshell. The real masters may also use vim ofcourse!
&nbsp;


Requirements: &nbsp;

Create a deployment with 2 pods. &nbsp;

Create a services with type LoadBalancer and re-use your public ip adress. &nbsp;

Expose the application on port 8080



&nbsp;
### The End
Well that's about it. Raise your hand if you have questions!
