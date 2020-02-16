# Imperative kubectl
Shout instructions

### Create a deployment
```bash
# Create a deployment from a demo nginx hello world image
kubectl create deployment --image nginxdemos/hello web1
deployment.apps/web1 created
```
### View your deployment
View your deployments with `kubectl get deployments` or `kubectl get deploy` for short.
```bash
kubectl get deploy                        
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

### view pods in your deployments
You can see the individual pods that make up your deplyoment through issuing `kubectl get pods`
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

### expose a deployment to the network
Make ports available by using `kubectl expose deployment <deployment name> --port=<port> --type=<type>` <br>
Adding load onto the load balancer will cause it to distribute the load on to different pods.
```bash
# Simple LoadBalancer example on port 80
kubectl expose deployment web1 --port=80 --type=LoadBalancer
service/web1 exposed
```
There are many types of ways to expose network ports, not only a LoadBalancer.

### view your exposed services
You can view your exposed services by issuing the command `kubectl get service <service name>`
```bash
kubectl get service web1
NAME   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
web1   LoadBalancer   10.102.40.76   <pending>     80:30067/TCP   131m
```

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

# Decalarative kubectl
Tell it what you want the end state to look like
Point to manifests (collection of files or a single file)
```bash
kubectl deploy -f <manifest file>
```
## kustomize
When you want to store a secret, you can amend your deployment?
