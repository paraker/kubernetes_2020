# probes
`probes` allow you to customise how kubernetes determines the status of your containers.<br>
`probes` can _run a command_, or _make a http request_.

## liveness probe
Indicates whether the container is running properly.<br>
Governs when the cluster will automatically stop or restart the container.<br>
liveness probes could catch a deadlock, where an application is running, but unable to make progress.<br>

#### exec liveness
Runs a `command`.
If the command succeeds, it returns 0, and the kubelet considers the Container to be alive and healthy.<br>
If the command returns a non-zero value, the kubelet kills the Container and restarts it.

```yaml
# runs a cat command on a file. If it succeeds, liveness is good.
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5  # wait time before first probe
      periodSeconds: 5  # running interval
```

**NOTE** 
For the first 30 seconds of the Container’s life, there is a /tmp/healthy file.<br>
So during the first 30 seconds, the command cat /tmp/healthy returns a success code.<br>
After 30 seconds, cat /tmp/healthy returns a failure code

#### httpGet
`httpGet probe` sends an HTTP GET request to the server (technically it sends it through the worker node's `kubelet`).<br>
In our example, that's a request on port `8080`.<br>
If the handler for the server’s `/healthz` path returns a success code, the `kubelet` considers the Container to be alive and healthy.<br>
If the handler returns a failure code, the `kubelet` kills the Container and restarts it.<br>

Any code greater than or equal to 200 and less than 400 indicates success. Any other code indicates failure.
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```
**NOTE** this example pod's web server will return successful http responses for 10 seconds only and will then deny the request.<br>
As such, your pod will keep restarting every 10 seconds.

#### tcpSocket
Pretty much like a netcat test.<br>
If the tcp socket succeeds, the check is passed.<br>
The example here contains a readiness probe as well, just to make the test pass.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

## readiness probe
Indicates whether the container is ready to service requests.<br>
Governs whether requests will be forwarded to the pod.<br>
When a Pod is not ready, it is removed from Service load balancers.<br>

An application might need to load large data or configuration files during startup, or depend on external services after startup.<br>
In such cases, you don’t want to kill the application, but you don’t want to send it requests either. <br>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-readiness-probe
spec:
  containers:
    - name: myapp-container
      image: nginx
      readinessProbe:  # 200-399 http status code means OK. Anything else is not.
        httpGet:  # do a http get request
          path: /  # root path on the container
          port: 80  # on port 80
        initialDelaySeconds: 5  # wait before first check
        periodSeconds: 5  # interval between checks
```
Until the readiness probe is succesful, our container will show READY 0/1!
```
kubectl get pod
NAME                 READY   STATUS    RESTARTS   AGE
my-readiness-probe   0/1     Running   0          4s
```

## startup probe
The kubelet uses `startup probes` to know when a Container application has started.<br>
If such a `probe` is configured, it disables `liveness` and `readiness` checks until it succeeds.<br>
Making sure those `probes` don’t interfere with the application startup.<br>
This can be used to adopt `liveness checks` on slow starting containers, avoiding them getting killed by the kubelet before they are up and running.

# container logging
logs are available to us through the normal docker container style syntax `kubectl logs ${pod}`.<br>

```yaml
# example pod that logs every second
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args: [/bin/sh, -c, 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']  # generates a log entry every second
```

## view logs
Of course, just list your logs with `kubectl logs <pod name>`
```
kubectl logs counter
0: Sun Mar  1 04:25:19 UTC 2020
1: Sun Mar  1 04:25:20 UTC 2020
2: Sun Mar  1 04:25:21 UTC 2020
3: Sun Mar  1 04:25:22 UTC 2020
```

## logs in a multi-container environment
Since you have multiple containers in the same pod, you must now specify which container to run `logs` commands on.<br>
Do this with the `--container` or `-c` flag for short.

```
# just an example, we don't have multi-container pods now
kubectl logs counter --container my-sidecar-counter /bin/sh
# /
```