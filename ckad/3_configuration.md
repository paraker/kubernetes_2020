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