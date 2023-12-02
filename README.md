# Helm introduction

## Setup Kubernetes playground

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

Click Add new instance twice and enter previously copied kubeadm join command on node2 and node3

To install helm:

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod +x get_helm.sh
export VERIFY_CHECKSUM=false
./get_helm.sh
```

Verify helm installed successfully with below
```
helm version
```

## Create helm chart

```
helm create nginx_demo
```

This will create default directory structure with below output

```
[node1 nginx_demo]$ ls -R
.:
Chart.yaml  charts  templates  values.yaml

./charts:

./templates:
NOTES.txt     deployment.yaml  ingress.yaml  serviceaccount.yaml
_helpers.tpl  hpa.yaml         service.yaml  tests

./templates/tests:
test-connection.yaml
[node1 nginx_demo]$ 
```

Chart.yaml would have below values after removing comments and empty lines
```
[node1 nginx_demo]$ egrep -v "^#|^$" Chart.yaml 
apiVersion: v2
name: nginx_demo
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.16.0"
[node1 nginx_demo]$ 
```

Remove all the template files and we will create templates maually.

```
rm -rf templates/*
k create deploy nginx-deployment --image nginx --replicas 2 --port=80 --dry-run=client -o yaml > templates/deployment.yaml
```