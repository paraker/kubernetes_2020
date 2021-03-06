# labels
`labels` are key-value pairs.<br>
These are attached to `primitives (objects)`.<br>
`labels` are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users.<br>

`Labels` can be used to organize and to select subsets of objects.<br>
`Labels` can be attached to objects at creation time and subsequently added and modified at any time.<br>
Each object can have a set of key/value labels defined. Each Key must be unique for a given object.<br>

`Labels` allow for efficient queries and watches and are ideal for use in UIs and CLIs.<br>
Non-identifying information should be recorded using `annotations`.


Deployments are often multi-dimensional, as such, these are examples of common labels:
* `"release" : "stable"`, `"release" : "canary"`
* `"environment" : "dev"`, `"environment" : "qa"`, `"environment" : "production"`
* `"tier" : "frontend"`, `"tier" : "backend"`, `"tier" : "cache"`

`labels` have a restricted set of characters that can be used. Like `a-z-_.`

## create labels in manifests
Simply add labels to the metadata in the pod manifest.
```yaml
# prod label pod
apiVersion: v1
kind: Pod
metadata:
  name: my-production-label-pod
  labels:  # labels block
    app: my-app  # label for "app"
    environment: production  # label for "environment"
spec:
  containers:
  - name: nginx
    image: nginx

# dev label pod
apiVersion: v1
kind: Pod
metadata:
  name: my-production-label-pod
  labels:  # labels block
    app: my-app  # label for "app"
    environment: development  # label for "environment"
spec:
  containers:
  - name: nginx
    image: nginx
```

#### Show your labels
Do this with `kubectl describe ${pod_name} --show-labels`.<br>
There is a section called `Labels`.

## find resources based on labels.
See `label selectors` section

# label selectors
Via a `label selector`, the client/user can identify a set of objects.<br>
The `label selector` is the core grouping primitive in Kubernetes.<br>
Use the `kuebctl get {primitive} --selector <selector>=<value>`<br>

`label selectors` can be stacked with a comma-separated syntax.<br>
**NOTE** when you stack labels, the first label is first used to filter, second then filters on the current result, etc.<br>
i.e. it's not an `or` or `and` going on here, it's just further filtering.

## equality-based selector
use `=` or `!=` to select resources.

```yaml
environment = production  # match anything equal to environment=production
tier != frontend  # match on "tier" is not "frontend"
environment=production,tier!=frontend  # match on production but not frontend, obviously.
```

Our example, select only pods with `environment` set to `production`
```
kubectl get pods -l environment=production
NAME                      READY   STATUS    RESTARTS   AGE
my-production-label-pod   1/1     Running   0          15m
```

## set-based selector
use `in`, `notin` and `exists (only the key identifier)`.


```yaml
environment in (production, qa)  # selects all resources with key equal to "environment" and value equal to "production" or "qa"
tier notin (frontend, backend)   # selects all resources with key equal to "tier" and values other than "frontend" and "backend," and all resources with no labels with the "tier" key.
partition                        # selects all resources including a label with key "partition"; no values are checked
!partition                       # selects all resources without a label with key "partition"; no values are checked
```

Our example, select only pods with `environment` in `production`
**NOTE** we have to use quotes around the set-based selectors as they contain spaces.
```
kubectl get pods -l 'environment in (production, development)'
NAME                       READY   STATUS    RESTARTS   AGE
my-development-label-pod   1/1     Running   0          18m
my-production-label-pod    1/1     Running   0          19m         16m
```

## combined selectors
You can combine both set-based and equality-based selectors.<br>

For example, like this:
`partition in (customerA, customerB),environment!=qa`

# annotations
`annotations` are very similar to labels with one clear distinction; they can't be used with selectors.<br>
`annotations` are simply arbitrary meta-data that can be attached to objects.<br>
This meta-data is intended for tools and libraries.<br>

`annotations` can be used to modify behaviour or engage non-standard features.<br>
Or they can be used to point logging, include build info, configs etc.

`annotations` can include characters not permitted in labels.

## create annotations in manifests
Just like with `labels`, you set `annotations` in the `metadata` block.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-annotation-pod
  annotations:  # annotation block
    owner: terry@linuxacademy.com  # arbitrary annotation info
    git-commit: bdab0c6  # arbitrary annotation info
spec:
  containers:
  - name: nginx
    image: nginx
```

As usual, show your `annotations` with `kubectl describe ${object_type} ${object_name}`

# deployments
A `deployment` declares the state of `pods` and `ReplicaSets`.<br>
`Deployments` are used for reliability, load-balancing and scaling.<br>
You describe a desired state in a `Deployment`, and the `Deployment Controller` changes the actual state to the desired state at a controlled rate.<br>  

`Rolling updates` is another feature of `deployments`.<br>

`deployments` are part of the `apps api`, which as such must be reflected in the yaml manifest.

## create deployment with manifest
Something like the below can be used to create a deployment.<br>

```yaml
apiVersion: apps/v1  # deployments use the apps/v1 api
kind: Deployment  # mandatory kind
metadata:  # mandatory metadata
  name: nginx-deployment
spec:
  replicas: 3       # count of pods replicas
  selector:         # The selector field defines how the Deployment finds which Pods to manage
    matchLabels:    # will match on pod labels
      app: nginx    # match any pods with label "app" = "nginx". Important, must match the template labels!
  template:         # desired state for pods in this deployment
    metadata:       # metadata to be passed to pods
      labels:       # labels that will be passed to pods
        app: nginx  # set label "app": "nginx" for all pods. Important for the selector!
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

Use `kubectl create/get/describe` as usual to create display and view your deployments.<br>

## edit deployments
simply use the `kubectl edit deployment <deployment name>`.<br>
This gives you an interactive way to for example increase or decrease your `replicas`.
The `Deployment Controller` will take care of the job for you once you declared what you want.

# rollouts and rollbacks
`rolling updates` provide a way to update a deployment to a new container version by gradually updating
replicas so that there is no downtime.

## rollouts with kubectl
`kubectl set image` is the command to be used for imperative action.<br>
This is how can be executed in a basic form `kubectl set image ${deployment_name} ${image_name}=${image}:${version}`<br>
`--record` can be used to record information about the update so that it can be rolled back later.<br>
Not 100% sure about what information, but at least the `--record` adds information to the `CHANGE-CAUSE` field in the `kubectl rollout history` output.

```
# update from nginx:1.7.1 to nginx:1.7.9
kubectl set image deployment/rolling-deployment nginx=nginx:1.7.9 --record
deployment.apps/rolling-deployment image updated
```

## get status of rollout
You can get the status of the rollout by executing `kubectl get rollout status deployment/${deployment_name}`

## history of update revisions
We can check revisions with `kubectl rollout history`.<br>
In this case we have three revisions. One of them (revision 5) has a failed image, nginx:1.8.5 doesn't exist.
```
kubectl rollout history deployment/rolling-deployment
deployment.apps/rolling-deployment 
REVISION  CHANGE-CAUSE
3         kubectl set image deployment/rolling-deployment nginx=nginx:1.7.1 --record=true
4         kubectl set image deployment/rolling-deployment nginx=nginx:1.7.9 --record=true
5         kubectl set image deployment/rolling-deployment nginx=nginx:1.8.5 --record=true
```

You can get details about a particular revision by specifying `revision=<revision>`<br>
Let's do that for our failing image revision, number 5.

```
kubectl rollout history deployment/rolling-deployment --revision=5
deployment.apps/rolling-deployment with revision #5
Pod Template:
  Labels:	app=nginx
	pod-template-hash=8687d6cfdd
  Annotations:	kubernetes.io/change-cause: kubectl set image deployment/rolling-deployment nginx=nginx:1.8.5 --record=true
  Containers:
   nginx:
    Image:	nginx:1.8.5
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```
## rollback (rollout undo) with kubectl 
We can easily rollback if our deployment update failed for whatever reason.<br>
We can choose a particular revision with `--to-revision=<revision number>`<br>
With the command simply executed by itself without the `--to-revision` flag, it will simply roll back one revision backwards.
```
# specific revision rollback, let's go back to nginx:1.7.9 that was working
kubectl rollout undo deployment/rolling-deployment --to-revision=4
deployment.apps/rolling-deployment rolled back
```

## yaml manifest rollout strategy config
The yaml manifest has a `strategy` section.<br>
These are the default values that came by us not specifying anything in our deployment manifest.<br>
Let's have a look at two specific values in the `strategy` section, namely `maxSurge` and `maxUnavailable`.<br>
These allow you to have fine-grained control during your rolling updates.
```yaml
spec:
  progressDeadlineSeconds: 600
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%  # This allows the replica count to go beyond the replica value set in the deployment during an update.
      maxUnavailable: 25%  # max number of replicas that can be unavailable during an udpate
    type: RollingUpdate
```

# jobs - one-off executions in pods
`jobs` can be utilised to run a workload until it completes.<br>
The `job` will create one or more `pods`. <br>
When the `job` is completed the container will exit and the `pod` will enter the `completed` state.<br>

This will create a simple job running a perl container that prints our a large number of pi decimals.
```yaml
apiVersion: batch/v1  # jobs are part of the batch api
kind: Job  # mandatory kind
metadata:  # mandatory metadata
  name: pi  # name our job
spec:  # mandatory spec
  template: # Desired state for pods in this job. Much like the deployment template.
    spec:
      containers:
      - name: pi
        image: perl  # run a perl container
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]  # print pi or whatever
      restartPolicy: Never  # never restart, just execute the job once.
  backoffLimit: 4  # how many times it will re-attempt if it would fail.
```

This is as normal deployed as a file with `kubectl create -f <yaml file>`.<br>

## view job and pods
List `job` like this.
```
kubectl get job
NAME   COMPLETIONS   DURATION   AGE
pi     1/1           27s        3m55s
```

Since the `job` actually runs a pod we can list `logs` and see how the `pod` finishes like it's supposed to.
```
kubectl get pods
NAME                                  READY   STATUS      RESTARTS   AGE
pi-nnj9x                              0/1     Completed   0          38s
```

# cronjobs - scheduled jobs
`cronjobs` are just like what they sound like, scheduled `jobs`.<br>
`cronjobs` are part of the batch/v1beta1 api. 

This is how a basic cronJob is defined in a manifest.
```yaml
apiVersion: batch/v1beta1  # mandatory api version
kind: CronJob  # mandatory kind
metadata: # mandatory metadata
  name: hello  # just set a name
spec:  # mandatory spec
  schedule: "*/1 * * * *"  # cron expression
  jobTemplate:  # desired state for job
    spec:  # job spec
      template: # nested job template, desired state for pods in this job
        spec:  # container spec
          containers:
          - name: hello # name pod hello
            image: busybox  # run busybox container
            args:  # run these arguments to busybox
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure  # restart if it fails
```

## view cronjobs and pods
View your `cronjobs` like normal with kubectl get.
```
# cronJob has not run yet, LAST SCHEDULE is <none>
kubectl get cronjobs
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        <none>          38s

# cronJob has now run 60 seconds ago
kubectl get cronjobs
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        60s             2m42s
```

This is a good time to view the `pods` that should have been created by the `cronjob`.
```
# a pod is created and completed for every cronjob execution.
 kubectl get pods
NAME                                  READY   STATUS      RESTARTS   AGE
hello-1583227620-l7pxs                0/1     Completed   0          2m27s
hello-1583227680-vfsdr                0/1     Completed   0          87s
hello-1583227740-rxx54                0/1     Completed   0          27s
```

# horizontal pod autoscaler
You can automgically scale deployments based on cpu usage or otherwise.
Create a `hpa` with something like this: `kubectl autoscale deployment mydeployment --max=10 --min=5 --cpu-percent 80`

## display your hpa

```
kubectl get hpa
NAME           REFERENCE                 TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
mydeployment   Deployment/mydeployment   <unknown>/80%   5         10        5          8m52s
```

## resulting hpa yaml 
```yaml
apiVersion: v1
items:
- apiVersion: autoscaling/v1
  kind: HorizontalPodAutoscaler
  metadata:
    annotations:
      autoscaling.alpha.kubernetes.io/conditions: '[{"type":"AbleToScale","status":"True","lastTransitionTime":"2020-03-07T06:56:37Z","reason":"SucceededGetScale","message":"the
        HPA controller was able to get the target''s current scale"},{"type":"ScalingActive","status":"False","lastTransitionTime":"2020-03-07T06:56:37Z","reason":"FailedGetResourceMetric","message":"the
        HPA was unable to compute the replica count: unable to get metrics for resource
        cpu: unable to fetch metrics from resource metrics API: the server is currently
        unable to handle the request (get pods.metrics.k8s.io)"}]'
    creationTimestamp: "2020-03-07T06:56:21Z"
    name: mydeployment
    namespace: default
    resourceVersion: "367585"
    selfLink: /apis/autoscaling/v1/namespaces/default/horizontalpodautoscalers/mydeployment
    uid: 5165bfcb-e0cc-463d-bbe5-2d6170e29b14
  spec:
    maxReplicas: 10
    minReplicas: 5
    scaleTargetRef:
      apiVersion: apps/v1
      kind: Deployment
      name: mydeployment
    targetCPUUtilizationPercentage: 80
  status:
    currentReplicas: 5
    desiredReplicas: 0
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""

```