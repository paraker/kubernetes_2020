# Architecture
## Master node
The master node contains four functions.<br>
1. api (for 
1. scheduler
1. command?
1. etcd

![master node](pictures/k8s_master.png =250x)

## node (worker node/minion)
Runs `kubelet` (communicates with the api on master) and `k-proxy`.<br>
The nodes exist for one purpose, to run pods.<br>
![node](pictures/node.png)
