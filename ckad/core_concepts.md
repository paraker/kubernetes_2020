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
```
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
*Note* you can't edit all values on a running pod. Some edits require deletion and recreation of the pod.
```
kubectl apply -f my-pod.yml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
```
You can also edit a pod with an interactive fast command `kubectl edit pod <pod name>.`<br>
Changes here will be applied immediately so there's no need to run an `apply` again.<br>
Same deal, you can't edit all fields in a running container!
```
# interactive edits
kubectl edit pod my-pod
```

## delete pods
You can delete a pod like this. This is required with some field edits in the pods.
```
kubectl delete pod my-pod
```