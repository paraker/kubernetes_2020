# Certified Kubernetes Application Developer (CKAD)

### Cloud server installation
[installation instructions](https://github.com/paraker/kubernetes_2020/blob/master/ckad/install_cloud.md) 

### Core concepts
[core concepts readme](https://github.com/paraker/kubernetes_2020/blob/master/ckad/core_concepts.md)
* pods
* namespaces
* basic container config

### configuration
[configuration readme](https://github.com/paraker/kubernetes_2020/blob/master/ckad/configuration.md)
* ConfigMaps
* SecurityContexts
* Resources requirements
* Secrets
* ServicesAccounts

### mutli-container pods
[multi-container pods readme](https://github.com/paraker/kubernetes_2020/blob/master/ckad/multi_container_pods.md)

### observability
[observability readme](https://github.com/paraker/kubernetes_2020/blob/master/ckad/observability.md)
* Liveness and Readiness Probes
* Container Logging
* Installing Metrics Server
* Monitoring Applications
* Debugging 

### pod design
[pod design readme](https://github.com/paraker/kubernetes_2020/blob/master/ckad/pod_design.md)
* Labels, Selectors, and Annotations
* Deployments
* Rolling Updates and Rollbacks
* Jobs and CronJobs 

### services and netoworking
[services and networking readme](https://github.com/paraker/kubernetes_2020/blob/master/ckad/services_and_networking.md)
* services
* NetworkPolicies

### state persistence
[state persistence readme](https://github.com/paraker/kubernetes_2020/blob/master/ckad/state_persistence.md)
* volumes
* PeristentVolumes and PersistenVolumeClaims

# kubernetes fundamentals
## install 
[instructions for kubectl, minikube and autocompletion](https://github.com/paraker/kubernetes_2020/blob/master/fundamentals/install_local.md)

## basics
[basics readme](https://github.com/paraker/kubernetes_2020/blob/master/fundamentals/basics.md)

Covers:
* pods
* services
* Controllers (ReplicaSets, daemonsets, statefulsets, deployments)
* namespaces
* kustomize
* imperative vs declarative kubectl
* Persistent Storage
* Update Applications (rolling updates)

## Administering Kubernetes
[admin readme](https://github.com/paraker/kubernetes_2020/blob/master/fundamentals/admin.md)
* Kubernetes dashboard
* Managing Resources
* Troubleshooting Pods and Nodes
* Understanding kubeadm