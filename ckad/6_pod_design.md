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

## create labels in manifests

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