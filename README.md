# Setup Kubernetes playground

Log in to https://labs.play-with-k8s.com/

Click Add new instance
On node1 enter below commands
 1. Initializes cluster master node:
``` 
 kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16
```
 2. Initialize cluster networking:

``` 
kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
```

 3. configure kubectl
```
export KUBECONFIG=/etc/kubernetes/admin.conf
alias k=kubectl
```
Test below command to check progress so far
```
k get no
```
In response to command 1, a join command will be printed with below template
```
 kubeadm join 192.168.0.23:6443 --token <> --discovery-token-ca-cert-hash sha256:<>
```
Copy this from the output logs to be used in next instance for the next instance to be joined to this cluster

Click Add new instance
enter previously copied kubeadm join command on node2

Click Add new instance
enter previously copied kubeadm join command on node3


