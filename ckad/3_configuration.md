# ConfigMaps
A `ConfigMap` is a kubernetes `Object (API primitive)` that stores configuration data in a key-value format.<br>
This key-value config can be used to configure software running in a container, by referencing the `ConfigMap` in the `Pod spec`.<br>

## create ConfigMap
```yaml
apiVersion: v1  # mandatory api version
kind: ConfigMap  # specify object to be a ConfigMap
metadata:  # mandatory metadata
  name: my-config-map  # name to reference in pods!
data:  # mandatory data block for ConfigMap
  myKey: myValue  # key-value data
  anotherKey: anotherValue  # key-value data
```

List and verify your `ConfigMap` 
```
# Get your configmap
kubectl get configmaps my-config-map
NAME            DATA   AGE
my-config-map   2      12s

# display contents of your configmap
kubectl describe configmap my-config-map 
Name:         my-config-map
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
anotherKey:
----
anotherValue
myKey:
----
myValue
Events:  <none>
```

## reference ConfigMap
references to ConfigMaps can be done in multiple ways.

### mount as environment variable
Our first option is to mount the ConfigMap as environment variables to our containers.<br>
This is done with the following type of yaml.
```yaml
# container spec with environment variable loaded from configmap
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]  # utilise environment variable from config map
    env:  # specify environment variables
    - name: MY_VAR  # create a name of the environment variable
      valueFrom:  # grab the value from an external source,
        configMapKeyRef:  # refer to a our config map called my-config-map
          name: my-config-map  # specifying the config map's name
          key: myKey  # specify the key-value we want to set for our variable

# verification of enviromnent variable (sh -c echo $(MY_VAR) && sleep 3600)
kubectl logs my-configmap-pod 
myValue

# verification by getting a shell and look for the variable
kubectl exec --stdin --tty my-configmap-pod sh
/# env | grep -i var
MY_VAR=myValue
```

### mount as volume
We can also mount ConfigMaps through container volumes, such that the configmap is like any other file that exists in that container.<br>
```yaml
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo $(cat /etc/config/*) && sleep 3600"]  # sh -c echo $(cat /etc/config/*) && sleep 3600
    volumeMounts:  # fields for mounting a volume to this container
      - name: config-volume  # specify the volume name that you want to mount to the container
        mountPath: /etc/config  # specify the path to where the volume will be mounted
  volumes:  # fields for volume mounts
    - name: config-volume  # name the volume appropriately, this is for a configmap so we call it config-volume
      configMap:  # instruct k8s that we want to use configmap data in our volume
        name: my-config-map  # reference the name of the ConfigMap that we want to use

# verify output from sh -c echo $(cat /etc/config/*)
kubectl logs -f my-configmap-volume-pod
anotherValuemyValue

# verify mounting and availability of the volume
kubectl exec --stdin --tty my-configmap-volume-pod -- ls /etc/config
anotherKey  myKey

# verify value of file in configmap volume
kubectl exec --stdin --tty my-configmap-volume-pod -- cat /etc/config/anotherKey
anotherValue
```

