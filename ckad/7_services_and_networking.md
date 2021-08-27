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
* NodePort: Exposes the Service on every Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the NodePort Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`. This defaults to ports `30000-32767`.
* LoadBalancer: Exposes the Service externally using a cloud provider’s load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.
* ExternalName: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record 

## imperative service example
This is an example of how you can imperatively create a service with `kubectl`<br>
`kubectl expose deployment web1 --port=80 --type=LoadBalancer`

## services example with selector (labels) - expose ClusterIP
Let's use our nginx deployment and expose it to the rest of the cluster through a cluster IP
```yaml
# in this case we are using the deployment https://raw.githubusercontent.com/paraker/kubernetes_2020/master/ckad/deployments/my-nginx-deployment.yaml

apiVersion: v1  # mandatory apiversion
kind: Service  # mandatory kind
metadata:  # mandatory metadata
  name: my-service  # just a name
spec:  # mandatory spec
  type: ClusterIP  # ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType.
  selector:  # label, equality, set or combined selector, your choice I believe
    app: nginx  # label selector. matches any app labelled "nginx"
  ports:  # mandatory ports block for exposure
  - protocol: TCP
    port: 8080  # port to expose on the cluster ip in this case
    targetPort: 80  # port on the container, nginx runs on port 80 by default. See this in kubectl describe pod <name>
```

Apply this with `kubectl apply -f service.yaml`.<br>
This specification creates a new `Service object` named “my-service”, which targets TCP port 8080 on any Pod with the `app=nginx` label and exposes port 80 to the rest of the cluster.<br>
Kubernetes assigns this Service an IP address (sometimes called the “cluster IP”)<br>
Note that this in only reachable by the cluster itself.<br>
The controller for the Service selector continuously scans for Pods that match its selector (label in this case) and updates the `endpoint`.

You can view your exposed services by issuing the command `kubectl get service <service name>`

## View your services
```bash
kubectl get svc my-service 
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
my-service   ClusterIP   10.101.48.48   <none>        8080/TCP   2m9s
```

## view endpoints on pods
The `endpoints` is where the service hooks in to the pods.<br>
In our case, we have three replicas so we will have three `endpoints`.
Show these with `kubectl get endpoints <svc name>`
```
 kubectl get endpoints my-service 
NAME         ENDPOINTS                                      AGE
my-service   10.244.1.47:80,10.244.1.48:80,10.244.2.36:80   68s
```

## View details of your services
With the `kubectl describe` command you get a new depth of information about your environment.<br>
With this you can see the namespace, IPs, ports, etc etc
```bash
kubectl describe svc/my-service
Name:              my-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP:                10.101.48.48
Port:              <unset>  8080/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.47:80,10.244.1.48:80,10.244.2.36:80
Session Affinity:  None
Events:            <none>
```

# network policies
By default, all `pods` in a cluster can communicate with any other `pod`, and reach out to any available IP.<br>
So for example, the `pod` IP we see in `kubectl get pods <name> -o wide` can be accessed by other `pods`!<br>

`networkPolicies` let's us restrict and control access between your `pods`.<br>
Note how `multi-container pods` would still be able to access each other.<br>

## plugin to support network policies
In order for `networkPolicies` to work, we need a network plugin that supports that.<br>
`canal` is such a plugin. Download and install like this.
```
wget -O canal.yaml https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml
kubectl apply -f canal.yaml
```

## sample pods for testing
Use these pods for testing out that a pod has access to another pod by default.
```
kubectl create -f https://raw.githubusercontent.com/paraker/kubernetes_2020/master/ckad/networkpolicies/network-policy-secure-pod.yaml
kubectl create -f https://raw.githubusercontent.com/paraker/kubernetes_2020/master/ckad/networkpolicies/network-policy-client-pod.yaml
```

Test access with curl
```
# get IP of "secure pod"
kubectl get pods network-policy-secure-pod -o wide
NAME                        READY   STATUS    RESTARTS   AGE    IP            NODE                    NOMINATED NODE   READINESS GATES
network-policy-secure-pod   1/1     Running   0          118s   10.244.2.37   par3c.mylabserver.com   <none>           <none>

# connect to it with curl
# note how you can use internal DNS name here too for the target service!
kubectl exec network-policy-client-pod -- curl 10.244.2.37
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   312k      0 --:--:-- --:--:-- --:--:--  597k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## manifest for network policy
```yaml
# use pods in this very directory

apiVersion: networking.k8s.io/v1  # newtork policies is part of networking.k8s.io api
kind: NetworkPolicy  # mandatory kind
metadata:  # mandatory metadata
  name: my-network-policy  # just a name
spec:  # mandatory spec
  podSelector:  # block for selector. label, equality, set or combined selector, your choice I believe
    matchLabels:  # we choose label selector
      app: secure-app  # any label "app" matching "secure-app" value will be target
  policyTypes:  # specify which whitelist policies that should be applied
  - Ingress     # defines that ingress ACL is applied
  - Egress      # defines that egress ACL is applied too
  ingress:      # define our ingress ACL
  - from:       # define source to permit
    - podSelector:  # selector that can probably be label, equality, set or combined.
        matchLabels:  # we use label as selector
          allow-access: "true"  # if label "allow-access" is set to "true" the source pod will get access
    - ipBlock:  # we also select on IP addresses to be more specific
        cidr: 172.17.0.0/16
        except:
          - 172.17.1.0/24
    - namespaceSelector:  # and thirdly we select on namespace too! now we are super specific
        matchLabels:
          project: myproject  # namespaces can have labels too
    ports:
    - protocol: TCP
      port: 80  # allow traffic on this port
  egress:       # egress ACL definition
  - to:         # define destination to permit
    - podSelector:  # another pod selector. can probably contain label, equality, set or combined.
        matchLabels:  # we choose to match on destination pods' labels
          allow-access: "true"  # if destination pod has label "allow-access" set to "true" then access is granted.
    ports:
    - protocol: TCP
      port: 80  # permit outbound port 80 to pods with the above label
```

### show network policies
Show current policies with `kubectl get networkpolicies`
```
kubectl get networkpolicies
NAME                POD-SELECTOR     AGE
my-network-policy   app=secure-app   44s
```

### show network policy details
show network policy details with `kubectl describe networkpolicies <name>`

```
 kubectl describe networkpolicy
Name:         my-network-policy
Namespace:    default
Created on:   2020-03-03 11:30:25 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     app=secure-app
  Allowing ingress traffic:
    To Port: 80/TCP
    From:
      PodSelector: allow-access=true
    From:
      IPBlock:
        CIDR: 172.17.0.0/16
        Except: 172.17.1.0/24
    From:
      NamespaceSelector: project=myproject
  Allowing egress traffic:
    To Port: 80/TCP
    To:
      PodSelector: allow-access=true
  Policy Types: Ingress, Egress
```

## test pod to pod access again - with network policy attached
My tests failed here. I can still access the secure pod from the client pod.<br>
I think this may be because my plugin installation may have failed.<br>
At least I was met with this error.
```
error: unable to recognize "canal.yaml": no matches for kind "DaemonSet" in version "extensions/v1beta1"
```

Anyhow, you get the idea. Re-run the `kubectl exec network-policy-client-pod -- curl 10.244.2.37` command to test.<br>
It should fail at this stage.<br>
You can repair the connectivity by running `kubectl edit pod network-policy-client-pod` and a metadata `labels` field with `allow-access: "true"`.


