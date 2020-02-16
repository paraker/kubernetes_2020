# Monitor your kubernetes
## View simple info
#### deployments
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
#### pods
You can see the individual pods that make up your deplyoment through issuing `kubectl get pods <pod name>`
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

### services
You can view your exposed services by issuing the command `kubectl get service <service name>`
```bash
kubectl get service web1
NAME   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
web1   LoadBalancer   10.102.40.76   <pending>     80:30067/TCP   131m
```
### namespaces
Namespaces are logical grouping of resources in the cluster worker nodes.<br>
```bash
kubectl get namespaces       
NAME              STATUS   AGE
default           Active   6h2m
kube-node-lease   Active   6h2m
kube-public       Active   6h2m
kube-system       Active   6h2m
```

## View details
With the `kubectl describe` command you get a new depth of information about your environment.<br>
With this you can see the namespace
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
Tell it what you want the end state to look like
Point to manifests (collection of files or a single file)
```bash
kubectl deploy -f <manifest file>
```
## kustomize
When you want to store a secret, you can amend your deployment?
