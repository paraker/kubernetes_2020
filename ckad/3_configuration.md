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
```