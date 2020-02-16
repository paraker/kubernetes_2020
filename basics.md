# pods
A pod is a group of one or more containers (such as Docker containers), with shared storage/network, and a specification for how to run the containers.<br>
All containers within a pod share the same IP address and volumes.<br>
They can also communicate via Inter-Process Communication (such as using shared memory space).<br>

If containers exist in separate pods, they can't use the IPC communication and would have to speak to each other via the pods' separate IP addresses.

## display your pods
You can see the individual pods that make up your deployment through issuing `kubectl get pods <pod name>`
```bash
kubectl get pods     
NAME                    READY   STATUS    RESTARTS   AGE
web1-74c74989b5-2kkbv   1/1     Running   2          120m
web1-74c74989b5-2vhnx   1/1     Running   2          120m
web1-74c74989b5-4l4ss   1/1     Running   2          120m
web1-74c74989b5-8p65q   1/1     Running   2          120m
web1-74c74989b5-9vp8k   1/1     Running   3          120m
web1-74c74989b5-dzqwl   1/1     Running   2          120m
web1-74c74989b5-k7htc   1/1     Running   2          120m
--- 
```
You can customise your output by using `--output=` or `-o=`.<br>
One example is `--output=wide` which gives you for example your IP address and the node of that pod.

```bash
kubectl get po -o=wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
web1-74c74989b5-jmsvg   1/1     Running   0          49m   172.17.0.2   minikube   <none>           <none>
```

## details about your pods
You can display a great deal of details about your pods with `kubectl describe pod <pod name>`<br>
For example you can see their `namespace`, which `node` they are on, their IP etc.
```bash
kubectl describe pod web1-74c74989b5-2kkbv
Name:         web1-74c74989b5-2kkbv
Namespace:    default
Priority:     0
Node:         minikube/192.168.99.100
```

## logs from your pods
Just like in docker, you can tail the logs from a group of containers.<br>
You do this with `kubectl logs -f <name of pod>`
```bash
kubectl logs -f web1-74c74989b5-jmsvg                             

172.17.0.1 - - [16/Feb/2020:07:42:35 +0000] "GET / HTTP/1.1" 200 7240 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:72.0) Gecko/20100101 Firefox/72.0" "-"
```

# Controllers
## ReplicaSet (rs for short)
A `ReplicaSet’s` purpose is to maintain a stable set of `replica Pods` running at any given time. As such, it is often used to guarantee the availability of a specified number of identical `Pods`.<br>
In other words, the `rs` will make sure that new replicas of pods are spun up if others are killed for whatever reason.<br>

We used `rs` to create 20 web servers as an example with the imperative `kubectl scale deployment --replicas 20 web1`<br> 
However, it is recommended to use declarative `deployments` that will create `rs` for you automatically, rather than doing an imperative creation.<br>

### view your replicasets
```bash
kubectl get replicasets          
NAME              DESIRED   CURRENT   READY   AGE
web1-74c74989b5   20        20        20      6h24m
```

### view details about your replicasets
```bash
kubectl describe replicasets web1-74c74989b5 
Name:           web1-74c74989b5
Namespace:      default
Selector:       app=web1,pod-template-hash=74c74989b5
Labels:         app=web1
                pod-template-hash=74c74989b5
Annotations:    deployment.kubernetes.io/desired-replicas: 20
```

## DaemonSet
A `DaemonSet` ensures that all (or some) Nodes run a copy of a specific `pod`.<br>
This is great for example for `fluentd`, `datadog agent` etc.<br>
Things you want on all nodes in your cluster, like logging etc.

```bash
# example daemonset is available at:
kubectl apply -f https://k8s.io/examples/controllers/daemonset.yaml
```

## StatefulSet
`StatefulSet` is the workload API object used to manage stateful applications.<br>
Manages the deployment and scaling of a set of `Pods` , and provides guarantees about the ordering and uniqueness of these `Pods`.

Like a `Deployment`, a `StatefulSet` manages `Pods` that are based on an identical container spec.<br>
Unlike a `Deployment`, a `StatefulSet` maintains a sticky identity for each of their `Pods`.<br>
These `pods` are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

StatefulSets can come in handy when your apps need:
* Stable, unique network identifiers.
* Stable, persistent storage.
* Ordered, graceful deployment and scaling.
* Ordered, automated rolling updates.

## deployments
A `deployment` declares the state of `pods` and `ReplicaSets`.<br>
`Deployments` are used for reliability, load-balancing and scaling.<br>
You describe a desired state in a `Deployment`, and the `Deployment Controller` changes the actual state to the desired state at a controlled rate.<br>  

`Rolling updates` is another feature of `deployments`. 

### display your deployments
View your deployments with `kubectl get deployments <deployment name>` or `kubectl get deploy` for short and for all deployments.
```bash
kubectl get deploy web1                       
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web1   1/1     1            1           3m17s
```

You can interactively view your deployments with `kubectl get deploy --watch` or `-w` for short.<br>
This will automatically update the output.
```bash
kubectl get deploy --watch                        
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web1   1/1     1            1           3m50s
```

wide output is available with `--output=wide`. Useful to see images and labels (selector).
```bash
kubectl get deployments --output=wide
NAME   READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES             SELECTOR
web1   1/1     1            1           7h47m   hello        nginxdemos/hello   app=web1
```

### details about your deployments
You can get a bunch of good details about your deployments with `kubectl describe deployments <deployment name>`.<br>
Super useful for troubleshooting and understanding fully what's in your deployment.<br>
Such as the desired replicas, the docker image, the volumes etc.
```bash
kubectl describe deployments web1
Name:                   web1
Namespace:              default
CreationTimestamp:      Sun, 16 Feb 2020 12:00:53 +1100
Labels:                 app=web1
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=web1
Replicas:               20 desired | 20 updated | 20 total | 20 available | 0 unavailable
StrategyType:           RollingUpdate
```

### autoscale deployments
You can let your deployment controller `autoscale` a deployment for you.<br>
It will judge based on oad I suppose?.
This is done imperatively with the command `kubectl autoscale deployment <deployment name> --min=<min> --max=<max>`<br>
Not sure how this is done through `yaml`.

# services
Kubernetes Pods are mortal. They are born and when they die, they are not resurrected.<br>
If you use a Deployment to run your app, it can create and destroy Pods dynamically.<br>
Each Pod gets its own IP address, however in a Deployment, the set of Pods running in one moment in time could be different from the set of Pods running that application a moment later.

`Services` is an abstraction of a logical set of pods.<br>
`Services` defines a policy (technically an API endpoint) of how you can access these pods.<br>
So for pods to be able to talk to each other and maintain that connectivity, you need `services`.<br>

Also, for your own web connectivity for example to a web service, you need to expose the services to the outside world.<br>
If not, the pod's internal cluster ip address will never be reachable by your computer.<br>

## services example with labels
Suppose you have a set of Pods that each listen on TCP port 9376 and carry a `label` `app=MyApp`
```bash
# example services yaml file
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
This specification creates a new `Service object` named “my-service”, which targets TCP port 9376 on any Pod with the `app=MyApp` label and exposes port 80 to the outside world.<br>
Kubernetes assigns this Service an IP address (sometimes called the “cluster IP”)<br>
The controller for the Service selector continuously scans for Pods that match its selector (label in this case) and updates the `endpoint`.

You can view your exposed services by issuing the command `kubectl get service <service name>`

## View your services
```bash
kubectl get service web1
NAME   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
web1   LoadBalancer   10.102.40.76   <pending>     80:30067/TCP   131m
```

## View details of your services
With the `kubectl describe` command you get a new depth of information about your environment.<br>
With this you can see the namespace, IPs, ports, etc etc
```bash
kubectl describe service web1
Name:                     web1
Namespace:                default
Labels:                   app=web1
Annotations:              <none>
Selector:                 app=web1
Type:                     LoadBalancer
IP:                       10.102.40.76
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30067/TCP
Endpoints:                172.17.0.10:80,172.17.0.11:80,172.17.0.12:80 + 17 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

# namespaces
Namespaces are intended for use in environments with many users spread across multiple teams, or projects.<br>
For clusters with a few to tens of users, you should not need to create or think about namespaces at all.<br>
`default namespace` is where all your pods and services will end up unless otherwise specified.
```bash
kubectl get namespaces       
NAME              STATUS   AGE
default           Active   6h2m
kube-node-lease   Active   6h2m
kube-public       Active   6h2m
kube-system       Active   6h2m
```

To set the namespace for a current request, use the --namespace flag.
```bash
kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>
kubectl get pods --namespace=<insert-namespace-name-here>
```

You can permanently save the namespace for all subsequent kubectl commands in that context.

```bash
kubectl config set-context --current --namespace=<insert-namespace-name-here>
# Validate it
kubectl config view --minify | grep namespace:
```

# Imperative kubectl
Shout instructions at your cluster basically

### Create a deployment
```bash
# Create a deployment from a demo nginx hello world image
kubectl create deployment --image nginxdemos/hello web1
deployment.apps/web1 created
```

### Scale your deployment
You can scale your deployment by issuing `kubectl scale deployment --replicas <desired number> <deployment name>`<br>
```bash
kubectl scale deployment --replicas 20 web1            
deployment.apps/web1 scaled

# view your new 20 deployments slowly become ready
kubectl get deployments                         
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web1   2/20    20           2           7m48s
```

### expose a deployment to the network
Make ports available by using `kubectl expose deployment <deployment name> --port=<port> --type=<type>` <br>
Adding load onto the load balancer will cause it to distribute the load on to different pods.
```bash
# Simple LoadBalancer example on port 80
kubectl expose deployment web1 --port=80 --type=LoadBalancer
service/web1 exposed
```
There are many types of ways to expose network ports, not only a LoadBalancer.


### Minikube special to connect to your deployment's load-balancer
minikube runs in a virtual machine.<br>
That virtual machine has an internally natted address that we need to extract to be able to connect to our service.<br>
minikube that we run in this course has a command to get the load-balancer URL from your deployment and automatically start your default browser.<br>
This is done with `minikube service <service name>`
```bash
minikube service web1
|-----------|------|-------------|-----------------------------|
| NAMESPACE | NAME | TARGET PORT |             URL             |
|-----------|------|-------------|-----------------------------|
| default   | web1 |             | http://192.168.99.100:30067 |
|-----------|------|-------------|-----------------------------|
🎉  Opening service default/web1 in default browser...
```

# Declarative kubectl
Tell it what you want the end state to look like.<br>
Point to `manifests` (a metadata file for an accompanying collection of files or a single file).<br>
These files are either in `yaml` or in `json`.<br>
These are at runtime converted to `json` and sent to the `kubernetes api`.
```bash
kubectl deploy -f <manifest file or URL>
```
## deploy from a URL or file
Let's look at an example of deploying a wordpress blog by declaring state.
```bash
# example for deploying a redis master
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-master-deployment.yaml

# confirm that your deployment was created
kubectl get deployments
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
redis-master   1/1     1            1           13m

# Confirm that you pod was created
kubectl get pods   
NAME                            READY   STATUS    RESTARTS   AGE
redis-master-7db7f6579f-8zkq7   1/1     Running   0          13m

# tail logs from redis master
kubectl logs -f redis-master-7db7f6579f-8zkq7
```
## delete deployments
You can clean up the whole deployment with `kubectl delete deployment <deployment name>`<br>
```bash
kubectl delete deployment redis-master
deployment.apps "redis-master" deleted
```
If you want to delete multiple deployments at the same time you can point to their `label`.<br>
The label is found by using the `kubectl describe deployment <name>`

```bash
#Find the label
kubectl describe deployment redis-master
Labels:                 app=redis

# delete by label
kubectl delete deployment -l app=redis                                                     
deployment.apps "redis-master" deleted
```
Or you can delete the deployment with the same file that you deployed
```bash
kubectl delete -f https://k8s.io/examples/application/guestbook/redis-master-deployment.yaml
```

# kustomize
`kustomize` started out as a standalone tool for customisation of kubernetes objects through `kustomization files`.<br>
However, since `kubectl` version 1.14, `kubectl` can utilise the `kustomization files` too! <br>

## secrets and configmaps 
Kustomize can be used to generate resources - `kubernetes secrets` and `configmaps`


Create a file for password info
```bash
cat password.txt      
username=admin
password=secret
```
Create a kustmization file for a secret
```bash
cat kustomization.yaml 
secretGenerator:
- name: example-secret-1
  files:
  - password.txt
```
View what kustomize would produce with `kubectl kustomize <kustomize dir>`
```bash
kubectl kustomize ./  
apiVersion: v1
data:
  password.txt: dXNlcm5hbWU9YWRtaW4KcGFzc3dvcmQ9c2VjcmV0Cg==
kind: Secret
metadata:
  name: example-secret-1-t2kt65hgtb
type: Opaque
```
Actually execute the kustomization with `kubectl apply --kustomize <kustomize dir>` or just `-k` for short.
```bash
kubectl apply -k ./                                                                        
secret/example-secret-1-t2kt65hgtb created
```
Verify that your secret was created
```bash
kubectl get secrets        
NAME                          TYPE                                  DATA   AGE
example-secret-1-t2kt65hgtb   Opaque                                1      4m37s
```

## setting cross-cutting fields
By cross-cutting fields we mean configuration fields that span an entire `project`.<br>
Such as setting the same `label` across all resources in a `project`.<br>
Or set the same `namespace`, or `name-prefixes`.<br>
Let's have a look at an example of a normal deployment yaml file.
```bash
# base deployment file
cat deployment.yaml   
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```
We specify a kustomization file to set namespace, name prefix, suffix, label, annotation.<br>
We specify which resources that this should affect.
```bash
# kustomization file
cat kustomization.yaml      
namespace: my-namespace
namePrefix: dev-
nameSuffix: "-001"
commonLabels:
  app: bingo
commonAnnotations:
  oncallPager: 800-555-1212
resources:
- deployment.yaml
```
Let's look at the result. We issue `kubectl kustomize ./` to run `kustomize` in the pwd.
```bash
# Resulting file. Notice the name-prefix, suffix, label and annotation. 
kubectl kustomize ./
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    oncallPager: 800-555-1212
  labels:
    app: bingo
  name: dev-nginx-deployment-001
  namespace: my-namespace
spec:
  selector:
    matchLabels:
      app: bingo
  template:
    metadata:
      annotations:
        oncallPager: 800-555-1212
      labels:
        app: bingo
    spec:
      containers:
      - image: nginx
        name: nginx
```

# All api resources
If you want to see all available api resources you can get them with kubectl.<br>
Simply issue the command `kubectl api-resource` and you get a big list of resources.<br>
You can use this to find the shortnames for the resources as an example.

```bash
kubectl api-resources       
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
bindings                                                                      true         Binding
componentstatuses                 cs                                          false        ComponentStatus
configmaps                        cm                                          true         ConfigMap
endpoints                         ep                                          true         Endpoints
events                            ev                                          true         Event
limitranges                       limits                                      true         LimitRange
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
```