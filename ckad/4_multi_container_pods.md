# multi-container pods
mutli-container pods are simply multiple containers in the same manifest.<br>
It is usually a good practice to separate containers to their own pods.<br>
However, if you wish to create containers that work together as a single unit, you can do so.<br>

## shared resources for mutli-container pods
Containers in the same pod have access to the same:
* network (can access each other without exposure of ports!)
* storage volume (can access the same volumes)
* process namespace (can reach each others processes)

## common design patterns
### sidecar
Main container + sidecar<br>
The sidecar enhances the functionality of the main container.<br>
Say that you have a main container serving static web files.<br>
The sidecar can every couple of minutes check for dynamic files and add them to the main container.<br>

### ambassador
Capturing and translating network traffic before it hits the main container.<br>
So the ambassador can sit as an interceptor in the network before it hits the main container.<br>
That could be for example like a port translator, like a poor man's load-balancer I suppose?

see example at [proxy-pod](https://raw.githubusercontent.com/paraker/kubernetes_2020/master/ckad/pods/my-multicontainer-proxy-pod.yaml)

### adapter
An adapter would sit between the main container and another system that requires output from the main container.<br>
For example if you have a log analyser that requires logs in a specific output format.<br>
You can then use an adapter container that can change your log formats to fit the recipient.<br>
Say that you want to modify your logs before they reach prometheus or logstash or whatever, but your app doesn't directly output the required formats.<br>
So rather than re-designing your whole app you can add on an adapter, perhaps :).

### sample non-real-world
Just a silly way of showing how we put multiple containers in the same pod:
```yaml
apiVersion: v1  # mandatory api version
kind: Pod  # mandatory kind
metadata:  # mandatory metadata
  name: my-multicontainer-pod  # name of pod
spec:  # mandatory spec
  containers:  # container block
    - name: nginx  # first container
      image: nginx  # nginx container image
      ports:  # port exposure block
      - containerPort: 80  # expose port 80 through clusterIP, I think
    - name: busybox-sidecar  # second container, or 'sidecar'
      image: busybox  # busybox image
      command: ['sh', '-c', 'echo hello kubernetes && sleep 3600']  # just a sleep for a while command
```

## exec in a multi-container environment
Since you have multiple containers in the same pod, you must now specify which container to run `exec` commands on.<br>
Do this with the `--container` or `-c` flag for short.

```
kubectl exec -it my-multicontainer-pod -c buxybox-sidecar /bin/sh
# /
```



## links to real examples

* https://kubernetes.io/docs/concepts/cluster-administration/logging/#using-a-sidecar-container-with-the-logging-agent
* https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/
* https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/