# kubernetes api primitives (also called kubernetes objects)
### relevant Documentation
    https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/

### full reference of api primitives (objects)
We can get a full reference list of all available api primitives.
```
kubectl api-resources -o name
```

### information about api primitives (objects)
Every object has a `spec` and a `status`.<br>
`spec` = defines the desired state. This is provided by you.<br>
`status` = current state of the object. Provided by the kubernetes cluster.<br>

#### kubectl get
To get rendered output from the cli for a given object we use the `kubectl get $object_type $object_name`.<br>
To get pure yaml output from the cli we use `--output yaml`.<br>
Note how the yaml output is super helpful in displaying our `spec` and `status`!
```
#rendered output
k get node par1c.mylabserver.com 
NAME                    STATUS   ROLES    AGE   VERSION
par1c.mylabserver.com   Ready    master   58m   v1.17.0

# raw yaml output
k get node par1c.mylabserver.com --output yaml
spec:
  podCIDR: 10.244.0.0/24
  podCIDRs:
  - 10.244.0.0/24
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
status:
  addresses:
    - address: 172.31.35.53
      type: InternalIP
```

#### kubectl describe
Get condensed and rendered information and logs about a primitive (object) by using the famous `kubectl describe $object_type $object_name` command.
It doesn't contain the actual yaml `spec` and `status`. However, it contains a far more readable, rendered version of the spec and status.<br>
All information you get with `kubectl get $object_type $object-name -o yaml` is available in `kubectl describe $object_type $ojbect_name`.

# pods
## create pods
pods are on of the most essential objects in k8s. This is a basic yaml definition of a pod running a busybox container.<br>
Note how we specify mandatory fields `apiVersion`, `kind`, `metadata` and `spec`. Without these it won't work.
```yaml
# my-pod.yaml
apiVersion: v1  # api version to use with kubernetes
kind: Pod  # type of api primitive (object)
metadata:
  name: my-pod  # name given to our object
  labels:
    app: myapp-1  # not mandatory by good for filtering later
spec:  # we provide the spec to kubernetes!
  containers:
    - name: myapp-container
      image: busybox  # The container we will run
      command: ['sh', '-c', 'echo hello kubernetes && sleep 3600']  # no built in command for busybox, supply one
```
Create the pod with `kubectl create -f my-pod.yaml`.<br>
Check that our hello world command succeeded with:
```
k logs -f my-pod
hello kubernetes
```

## edit pods
Edit a pod by updating the yaml definition and re-applying it, this time with `apply`.<br>
*Note* how it will complain about editing the file the first time. You can ignore this.<br>
*Note* you can't edit all values on a running pod. Some edits require deletion and recreation of the pod.<br>
```
kubectl apply -f my-pod.yml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
```
You can also edit a pod with an interactive fast command `kubectl edit pod <pod name>.`<br>
Changes here will be applied immediately so there's no need to run an `apply` again.<br>
Same deal, you can't edit all fields in a running container!<br>
Examples of things you can't change interactively:
* namespace
```
# interactive edits
kubectl edit pod my-pod
```

## delete pods
You can delete a pod like this. This is required with some field edits in the pods.
```
kubectl delete pod my-pod
```

# namespaces
Not a part of the objectives for CKAD. But good to know about namespaces.<br>
At a high level namespaces are used when:
* multiple apps in a cluster. You may put apps in their own namespace
* multiple teams using the same cluster. They can have their own portion of the cluster

## get namespaces
See all namespaces with `kubectl get namespace`
```
kubectl get namespaces
NAME              STATUS   AGE
default           Active   113m
kube-node-lease   Active   113m
kube-public       Active   113m
kube-system       Active   113m
```

## create namespaces
create namespaces (ns) with `kubectl create ns <name>`
```
# create
kubectl create ns my-ns
namespace/my-ns created

# verify
kubectl get namespaces
NAME              STATUS   AGE
default           Active   113m
kube-node-lease   Active   113m
kube-public       Active   113m
kube-system       Active   113m
my-ns             Active   6s
```

## add object to namespace
When constructing your objects' yaml file, ensure that the `metadata` field to includes a `namespace` field.
```yaml
# my-ns-pod.yaml
apiVersion: v1  # api version to use with kubernetes
kind: Pod  # type of api primitive (object)
metadata:
  namespace: my-ns  # defines the namespace
  name: my-ns-pod  # name given to our object
  labels:
    app: myapp-1  # not mandatory by good for filtering later
spec:  # we provide the spec to kubernetes!
  containers:
    - name: myapp-container
      image: busybox  # The container we will run
      command: ['sh', '-c', 'echo hello kubernetes && sleep 3600']  # no built in command for busybox, supply one
```

Note that you can't run a `kubectl edit $object_type $object_name` when changing namespaces.<br>
You will end up with an error like this if you do `kubectl edit`
```
kubectl edit pod my-pod 
A copy of your changes has been stored to "/tmp/kubectl-edit-lny8j.yaml"
error: the namespace from the provided object "my-ns" does not match the namespace "default". You must pass '--namespace=my-ns' to perform this operation.
```

```
# create
kubectl create -f my-ns-pod.yaml

# verify
kubectl get pods -n my-ns
NAME        READY   STATUS    RESTARTS   AGE
my-ns-pod   1/1     Running   0          3m41s
```

## delete namespace
Delete your namespace with `kubectl delete ns my-ns`.<br>
**NOTE** this will delete all your objects in that namespace too!
```
# delete namespace and all objects in the namespace
k delete ns my-ns
namespace "my-ns" deleted
```

# basic container spec configs
Some frequently-used container configuration options that you may edit are: `command`, `args`, and `containerPort`.<br>

## command
command will modify the container CMD. This is great if your container doesn't have a CMD to start with, or if you want to modify the existing value.<br>
In the `spec` section of the yaml, add a `command` field the `container` field.

```
# add 
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['echo hello']  # run a simple echo command when container is fired up
  restartPolicy: Never  # do not attempt to restart the container when command is finished

# verify run. Should say "Completed"
kubectl get pods
NAME             READY   STATUS      RESTARTS   AGE
my-command-pod   0/1     Completed   0          6s

```

