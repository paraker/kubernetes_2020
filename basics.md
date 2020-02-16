# Imperative kubectl
Shout instructions

### Create a deployment
```bash
# Create a deployment from a demo nginx hello world image
kubectl create deployment --image nginxdemos/hello web1
deployment.apps/web1 created
```
### View your deployment
View your deployments with `kubectl get deploy`
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
kubectl get deploy                         
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web1   2/20    20           2           7m48s
```

### expose a deployment
Make ports available by using `kubectl expose deployment <deployment name> --port=<port> --type=<type>`
```bash
# Simple LoadBalancer example on port 80
kubectl expose deployment web1 --port=80 --type=LoadBalancer
service/web1 exposed
```

### Minikube special to connect to your deployment's load-balancer
minikube that we run in this course has a command to get the load-balancer URL from your deployment and automatically start your default browser. This is done with `minikube service <service name>`
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
