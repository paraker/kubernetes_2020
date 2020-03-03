# services
Kubernetes `Pods` are mortal. They are born and when they die, they are not resurrected.<br>
If you use a `Deployment` to run your app, it can create and destroy `Pods` dynamically.<br>
Each `Pod` gets its own IP address, however in a `Deployment`, the set of `Pods` running in one moment in time could be different from the set of Pods running that application a moment later.

`services` is an abstraction of a logical set of pods.<br>
`services` defines a policy (technically an `API endpoint`) of how you can access these pods.<br>
So for `pods` to be able to talk to each other and maintain that connectivity, you need `services`.<br>

Also, for your own web connectivity for example to a web service, you need to expose the services to the outside world.<br>
If not, the pod's `internal cluster ip` address will never be reachable by your computer.<br>

## ServiceTypes - four types of services
There are four types of services that you can create with kubernetes:
* ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType.
* LoadBalancer: Exposes the Service externally using a cloud provider’s load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.
* NodePort: Exposes the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the NodePort Service, from outside the cluster, by requesting <NodeIP>:<NodePort>.
* ExternalName: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record 

## imperative service example
This is an example of how you can imperatively create a service with `kubectl`<br>
`kubectl expose deployment web1 --port=80 --type=LoadBalancer`

## services example with selector (labels) - expose ClusterIP
Suppose you have a set of Pods that each listen on TCP port 9376 and carry a `label` `app=MyApp`
```yaml
# example services yaml file

```

Apply this with `kubectl apply -f service.yaml`.<br>
This specification creates a new `Service object` named “my-service”, which targets TCP port 9376 on any Pod with the `app=MyApp` label and exposes port 80 to the rest of the cluster.<br>
Kubernetes assigns this Service an IP address (sometimes called the “cluster IP”)<br>
Note that this in only reachable by the cluster itself.<br>
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