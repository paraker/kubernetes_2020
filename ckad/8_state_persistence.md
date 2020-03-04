# volumes
Containers have ephemeral storage by default, i.e. when they shut down the storage is lost.<br>
Stateful applications don't fare well with this approach so kubernetes offers a range of [persistent volume storage options](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes).<br>
This is useful for sharing storage between containers as well, even for stateless apps.<br>

## mounting volumes

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

You typically create persistent volumes with the declarative yaml approach.<br>
`PersistentVolume` (PV) are created by admins. 