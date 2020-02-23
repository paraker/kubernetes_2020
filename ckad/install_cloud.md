# install instructions in a cloud environment
## Server infra
spin up three servers with 2cpu and 4gb ram. Use ubuntu xenial 16.04 LTS

## installation commands
add repos to apt
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

install packages. Hold them (prevent them from updating).
```bash
sudo apt-get update

sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu kubelet=1.17.0-00 kubeadm=1.17.0-00 kubectl=1.17.0-00

sudo apt-mark hold docker-ce kubelet kubeadm kubectl
```

ensure proper network settings
```bash
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf

sudo sysctl -p
```


### On the Kube master server

#### Initialize the cluster:

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
#### Set up local kubeconfig:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
#### Copy command for joining nodes
note how the output from the kubeadm init command gave you a string like the below
```
kubeadm join 172.31.35.53:6443 --token biqxj1.2megiizxrqj3 \
    --discovery-token-ca-cert-hash sha256:2bdd6a14d3d280b68157de8d9dd957c4a469798dc5a1e0ed9139a7
```
Just make sure you have this handy later in this guide

#### Install Flannel networking:

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/3f7d3e6c24f641e7ff557ebcea1136fdf4b1b6a1/Documentation/kube-flannel.yml
```
### On each Kube node server

Join the node to the cluster:

```
sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash
```
On the Kube master server

Verify that all nodes are joined and ready:

```
kubectl get nodes
```
You should see all three servers with a status of Ready:

```
k get nodes
NAME                    STATUS   ROLES    AGE   VERSION
par1c.mylabserver.com   Ready    master   38m   v1.17.0
par2c.mylabserver.com   Ready    <none>   36m   v1.17.0
par3c.mylabserver.com   Ready    <none>   35m   v1.17.0
```

## autocompletion and alias for your bash

```
# enable autocompletion for kubectl
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

# add an alias to your bashrc to make kubectl just "k"
cat <<EOF >> ~/.bashrc
alias k=kubectl
complete -F __start_kubectl k
EOF
```