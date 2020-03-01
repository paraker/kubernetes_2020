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
Do this with `kubectl describe ${pod_name}`.<br>
There is a section called `Labels`.

## find resources based on labels.
See `selectors` section

# label selectors
Via a `label selector`, the client/user can identify a set of objects.<br>
The `label selector` is the core grouping primitive in Kubernetes.

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