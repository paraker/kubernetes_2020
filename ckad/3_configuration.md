# configMaps
A `configMap` is a kubernetes `Object (API primitive)` that stores configuration data in a key-value format.<br>
This key-value config can be used to configure software running in a container, by referencing the `configMap` in the `Pod spec`.<br>

## create configMap
```yaml
apiVersion: v1  # mandatory api version
kind: configMap  # specify object to be a configMap
metadata:  # mandatory metadata
  name: my-config-map  # name to reference in pods!
data:  # mandatory data block for configMap
  myKey: myValue  # key-value data
  anotherKey: anotherValue  # key-value data
```

List and verify your `configMap` 
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

## reference configMap
references to configMaps can be done in multiple ways.

### mount as environment variable
Our first option is to mount the configMap as environment variables to our containers.<br>
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
We can also mount configMaps through container volumes, such that the configMap is like any other file that exists in that container.<br>
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
        name: my-config-map  # reference the name of the configMap that we want to use

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

# secrets
A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key.<br>
Such information might otherwise be put in a Pod specification or in an image.<br>
Users can create secrets and the system also creates some built-in secrets.<br>

To use a secret, a Pod needs to reference the secret. A secret can be used with a Pod in two ways:
* As files in a volume mounted on one or more of its containers.
* By the kubelet when pulling images for the Pod

## create a secret manually from yaml
You can use the normal `yaml` file syntax to create secrets.<br>
There are two ways to include the sensitive data in these files; base64 encoded (`data`) or as normal strings (`stringData`).<br>

```yaml
# Sample stringData secret
apiVersion: v1  # mandatory api version
kind: Secret  # Object to store sensitive data
metadata:  # mandatory metadata block
  name: my-stringdata-secret # name to be referenced in pods/deployments!
stringData:  # write-only convenience block to not have to deal with base64 encoding
  myKey: myPassword  # configure a key-value pair as a simple string
```

Generally after the secret has been created in kubernetes, you should/can delete the yaml file that includes the sensitive information.<br>

## using secret from a pod spec

# SecurityContexts for Pods
a `securityContext` for a `Pod` defines its privilege and access control settings.<br>
If a pod requires special OS-level permissions, we can provide them via a `securityContext`.

In our example, we will have a file that is only readable by a particular user/group.<br>
This user/group has been created _on the workernodes_!

```yaml
spec:
  securityContext:  # fields for securityContext
    runAsUser: 2000  # run as this user id (defined by worker nodes OS!)
    fsGroup: 3000  # run as this group (defined by worker nodes OS!)
  containers:  # fields for containers
    - name: myapp-container
      image: busybox  # just a normal busybox container
      command: ['sh', '-c', "cat /message/message.txt && sleep 3600"]  # sh -c cat /message/message.txt && sleep 3600
      VolumeMounts:  # fields for mounting volumes to container
        - name: message-volume  # name of the volume we are mounting
          mountPath: /message  # mounts to /message on the container
  volumes:  # fields for volumes
    - name: message-volume  # mountable name for volume 
      hostPath:  # specifies that it's on the worker node (Host)
        path: /etc/message  # path on the worker node

#This should be done on your worker nodes
#sudo useradd -u 2000 container-user-0
#sudo groupadd -g 3000 container-group-0
#sudo useradd -u 2001 container-user-1
#sudo groupadd -g 3001 container-group-1
#sudo mkdir -p /etc/message/
#echo "Hello, World!" | sudo tee -a /etc/message/message.txt
#sudo chown 2000:3000 /etc/message/message.txt
#sudo chmod 640 /etc/message/message.txt

# verification that we use user "2000"
k exec my-securitycontext-pod -- ps
PID   USER     TIME  COMMAND
    1 2000      0:00 sleep 3600
    7 2000      0:00 ps

# verification that we could look at the file
cloud_user@par1c:~/ckad$ k logs my-securitycontext-pod 
Hello, World!
```

If we reconfigure the `pod spec` ot use `runAsUser: 2001` and `fsGroup: 3001` we can confirm that the approach works.<br>
It crashes because it can't read the file.
```
# test of a container running as user 2001 and group 3001
k logs my-securitycontext-pod 
cat: can't open '/message/message.txt': Permission denied

# pod gets stuck on CrashLoopBackOff
k get pods
NAME                      READY   STATUS             RESTARTS   AGE
my-securitycontext-pod    0/1     CrashLoopBackOff   1          26s
```

# resource requirements
kubernetes allows us to specify `resource requirements` in the `pod spec`.<br>
A container's memory and cpu requirements are defined in terms of `resource requests` and `limits`.<br>

**resource request** - the amount of resources necessary to run a container. A pod will only be able to run a container if it has enough spare resources for the resource request.<br>
**resource limit** - a maximum value for the resource usage of a container. If the container uses more than this, it's likely killed by kubernetes.

```yaml
# pod spec with resource requests and limits
spec:
  containers:
    - name: myapp-container
      image: busybox
      command: ['sh', '-c', 'echo Hello kubernetes! && sleep 3600']
      resources:  # fields for resources
        requests:  # minimum required resources for worker node to take on the pod
          memory: "64Mi" # 64 Mebibytes (mega + binary = mebi)
          cpu: "250m"  # 250 milliCPUs, i.e. 0.25 CPU cores
        limits:  # maximum resources used by the container, if more is used, probably kill it.
          memory: "128Mi"  # 128 Mebibytes
          cpu: "500m"  # 500 milliCPUs, 0.5 CPU cores

# verification through describe
kubectl describe my-resource-pod
---
    Limits:
      cpu:     500m
      memory:  128Mi
    Requests:
      cpu:        250m
      memory:     64Mi

```

# generating yaml files
A quick and easy way to generate a formatted yaml file for you is through `kubectl run`.<br>

```
# generate a deployment yaml file, ready to be edited by you
kubectl run my-sample-deploy --image busybox --dry-run -o yaml > my-sample-deploy.yaml
# note how certain field are left empty with {}. This does not mean they're maps, it just means they're empty, afaik.
```