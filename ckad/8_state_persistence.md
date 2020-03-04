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

## PVC (persistent volume claim)
A PersistentVolumeClaim (PVC) is a request for storage by a user. It is similar to a Pod.<br>
Pods consume node resources and PVCs consume PV resources.