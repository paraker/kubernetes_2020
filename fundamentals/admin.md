# Kubernetes dashboard
A graphical representation of your kubernetes cluster..<br>
Fairly neatly shows deployments, pods, controllers etc etc.<br>
You can modify, add, delete objects etc. Seems like you can do whatever you can in the cli.

## Install kubernetes dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
```

## launch in minikube
This is failing for me for some reason. No time to investigate that for now.
```buildoutcfg
➜  ~ minikube dashboard
🔌  Enabling dashboard ...

💣  Unable to enable dashboard: running callbacks: [addon apply: sudo KUBECONFIG=/var/lib/minikube/kubeconfig /var/lib/minikube/binaries/v1.17.2/kubectl apply -f /etc/kubernetes/addons/dashboard-ns.yaml 
-f /etc/kubernetes/addons/dashboard-clusterrole.yaml -f /etc/kubernetes/addons/dashboard-clusterrolebinding.yaml 
-f /etc/kubernetes/addons/dashboard-configmap.yaml -f /etc/kubernetes/addons/dashboard-dp.yaml 
-f /etc/kubernetes/addons/dashboard-role.yaml -f /etc/kubernetes/addons/dashboard-rolebinding.yaml -f /etc/kubernetes/addons/dashboard-sa.yaml 
-f /etc/kubernetes/addons/dashboard-secret.yaml -f /etc/kubernetes/addons/dashboard-svc.yaml: Process exited with status 1
stdout:

```

# get to a shell in a container
We can very much like in docker get a shell in a container. <br>
This is done interactively with `kubectl exec --stdin --tty <pod name> -- <command>`.<br>
or `kubectl exec -it <pod name> -- <command>` for short.<br>
I'm not really sure why the `--` suggested here. It works without the `--` too

```bash
# get a shell
kubectl exec -it nginx-6db489d4b7-zsrw2 -- /bin/bash

# execute a one-off command
kubectl exec nginx-6db489d4b7-zsrw2 cat /etc/hosts 
# Kubernetes-managed hosts file.
127.0.0.1	localhost
```

# Copy a pod config to a yaml file.
You can take an existing pod and get the full config out of it.<br>
This could be useful in development and testing scenarios.<br>
You do this by redirecting the output from teh `kubectl get pods` command with the `--output` flag. `-o` for short if you want.

```
# get the nginx pod config output in yaml format, redirect output to a file.
kubectl get pods nginx-6db489d4b7-zsrw2 --output yaml > nginx-resource-test.yaml
```

Now you can manually modify this file to your liking and create a new pod from it with `kubectl apply -f <file.yaml>`.<br>
Note how `pods` can't have the same name, so that's one field you must obviously modify.<br>
Basically I think you have to take out all of the metadata fields except `name` and `namespace`.<br>
And everything else down to the `spec`. This is how the top of my nginx file turned out.
```
# example of the top of a yaml file for nginx that has been copied from a running pod.
apiVersion: v1
kind: Pod
metadata:
  name: nginx-resource-test
  namespace: default
spec:
  containers:
  - image: nginx
```

# Managing resources
kubernetes allows you to control cpu and memory resources per container.<br>
API resources (whatever they are) can be modified through the k8s api server.<br>
CPU and memory resources are controlled by limits and requests. <br>
The scheduler then takes care of the limits and requests and makes sure the pods are deployed as such.<br>

## metrics server
The metrics server can be used to visualise and manage resources usage within the kubernetes cluster.
```
# enable the minikube metrics server
minikube addons enable metrics-server
🌟  The 'metrics-server' addon is enabled
```
We can now modify existing yaml files with additional resource fields to set `limits` and `requests.`<br>
The metrics server also enables resource monitoring commands such as `kubectl top node` and `kubectl top pod`.

## set a resource limitation in a yaml file
Let's look at an example of a resource limitation, the cpu

```
cat nginx-resource-test.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-resource-test
  namespace: default
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx
    resources:
            limits:
                    memory: "512Mi"
                    cpu: "1"
            requests:
                    cpu: "0.5"
```
Note that if this `pod` was already applied, you can't just re-apply the config with changed limits.<br>
This is not accepted by k8s. What you have to do is delete the `pod` and recreate it.<br>
kubectl delete pod/nginx-resource-test
```
# after resources changes we delete the pod
 kubectl delete pod/nginx-resource-test  
pod "nginx-resource-test" deleted

# and create the pod
 kubectl create -f nginx-resource-test.yaml 
pod/nginx-resource-test created
```
Then we have describe the pod to verify the limits and requests for resources.<br>
We can see that we indeed have limits set to 1 cpu and 512Mi memory.<br>
Since we didn't do a resource request for memory, it seems to have chosen the same value as the limit.
```
kubectl describe pod nginx-resource-test
---   
    Limits:
      cpu:     1
      memory:  512Mi
    Requests:
      # 500m cpu = 500 milli cpu = 0.5 cpu
      cpu:        500m
      memory:     512Mi
```

## monitor node resource usage
Use the `kubectl top node` command to see the resource usage on the top nodes<br>
We only have one node in our minikube install to show resources from<br>
If you scale up your deployments to have many replicas, naturally the resources usage should of course increase.
```
 kubectl top node                        
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
minikube   559m         27%    1423Mi          77%  
```

## monitor pod resource usage
Use the `kubectl top pod` command to see resource usage on the top pods.<br>
Note how these values do not seem to be instant on minikube! Quite a bit of delay in the output.<br>
Note how these values are for individual pods.
```buildoutcfg
kubectl top pods                               
NAME                            CPU(cores)   MEMORY(bytes)   
nginx-6db489d4b7-zsrw2          0m           3Mi             
nginx-resource-test             0m           2Mi 
```

# Troubleshooting pods
## pod conditions
pending 
running (actively executing)
succeeeded (started and did what it was supposed to do)
failed (one container in the pod failed)
unknown (k8s scheduler can't communicate with the pod)

## container conditions
waiting 
running
terminated

## troubleshooting commands
`kubectl describe <pod>` is really the go-to command.<br>
The events section is the most helpful section to give you an idea of where to continue.