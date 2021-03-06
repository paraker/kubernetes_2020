# volumes
Containers have ephemeral storage by default, i.e. when they shut down the storage is lost.<br>
Stateful applications don't fare well with this approach so kubernetes offers a massive range range of [volumes](https://kubernetes.io/docs/concepts/storage/volumes/), like 40 fo them.<br>
These can be like `EBS`, `hostPath`, `nfs`, `emptyDir` etc.
This is useful for sharing storage between containers as well, even for stateless apps.<br>
This is your direct way of attaching storage to a pod.

## mounting volumes
Let's mount a simple [`emptyDir` volume](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir).<br>
Sort of a node-specific volume that all pods on the node can access, I think. 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
    volumeMounts:  # enable volume mounting
    - mountPath: /tmp/storage  # mount volumes to this mount point
      name: my-volume  # mount the volume called "my-volume"
  volumes:  # list of volumes that we declare
  - name: my-volume  # we call this volume
    emptyDir: {}  # we create an emptydirectory type of volume. There are many types of volumes
```

# Persistent Volumes - and claims!
Managing storage is a distinct problem from managing compute instances.<br>
The PersistentVolume subsystem provides an API for users and administrators.<br>
This API abstracts details of how storage is provided from how it is consumed.<br>
This is a nice and robust abstraction layer that helps you mount storage.<br>
This is in contrast to the volumes that are mounted straight away on the pods, I think at least.<br>

We are introduced to two new API resources: `PersistentVolume` and `PersistentVolumeClaim`.
### types of persistent volumes
`PVs` can be `EBS`, `AzureDisk` etc etc.

### why is it good?
* `PVCs` can say for example "I want fast storage of this type" and automagically get that type of storage provisioned<br>
(It does so by using storageClasses) 

## PV (persistent volume)
A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.<br> 
You typically create persistent volumes with the declarative yaml approach.<br>

Let's create a noob version of storage, locally on one of the nodes.
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  storageClassName: local-storage  # Free text field. Call your storage class something arbitrary like "Fast", "Slow", "local", etc.
  capacity:  # capacity block
    storage: 1Gi  # 1 gibibyte
  accessModes:  # different type of storage permits different types of access modes
    - ReadWriteOnce  # only one pod at a time can read/write
  hostPath:  # (Single node testing only – local storage is not supported in any way and WILL NOT WORK in a multi-node cluster)
    path: "/mnt/data"
```
### list PV
```
kubectl get pv
NAME    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    REASON   AGE
my-pv   1Gi        RWO            Retain           Available           local-storage            5s
```

## PVC (persistent volume claim)
A PersistentVolumeClaim (PVC) is a request for storage by a user. It is similar to a Pod.<br>
Pods consume node resources and PVCs consume PV resources.<br>

So as a developer you can make a `claim` that you can later map to `pods`.<br>
Let's make a claim for 512 Mebibytes.<br>

We declare and then the system will hand out storage if it's available.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: local-storage  # Important field! This is referring to the class name of the local storage!
  accessModes:
    - ReadWriteOnce  # this should also be the same as the PV
  resources:  # resource limits for our request
    requests:  # this is our request going to the PV
      storage: 512Mi  # We request 512 Mebibytes. We know 1 Gibibyte is available
```

### list pvc
Great, our claim was successful.<br>
The admins must have provisioned enough storage, awesome.
```
 kubectl get pvc
NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS    AGE
my-pvc   Bound    my-pv    1Gi        RWO            local-storage   5s
```

## Use PVC in a volume in a pod
Now it's time to use the PVC in our pod.<br>
This is just like a static volume mount on the pod `volumeMount`, but the `volume` part is different.
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: my-pvc-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
    volumeMounts:  # Mount volumes to this pod
    - mountPath: "/mnt/storage"  # this is where to mount it to
      name: my-storage  # refer to the storage
  volumes:  # create volumes for this pod
  - name: my-storage  # call volume "my-storage"
    persistentVolumeClaim:  # Use our previously created PVC
      claimName: my-pvc  # use the claim called "my-pvc". Refer to the metadata section of the PVC
```

### describe pod volume
We can see how the pod is consuming the volume coming from a volume claim through `kubectl describe ${pod_name}`
```
Volumes:
  my-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  my-pvc
    ReadOnly:   false
  default-token-jq77c:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-jq77c
    Optional:    false

```
