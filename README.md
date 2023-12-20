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

**Terminology**:

Chart: Collection of files that describes a set of kubernetes resources including templates for YAML files, default configuration values and metadata (name, version of chart etc)

Release: An instance of a chart running in Kubernets cluster which describes a specific versin of chart having unique name and set of configuration options. This is the name that you provide while instantiating a chart.

Lets templatize the deployment file we created above for below items

After templating release name, replicas, container name, image and imagePullPolicy the final deployment tempalte will look like below:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-nginx
  labels:
    app: nginx
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: {{ .Release.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
```

Create a service:

```
k create service clusterip nginx-service --tcp 80 --dry-run=client -o yaml > templates/service.yaml
```

After templating the service, the final service.yaml will look like below:

```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: {{ .Release.Name }}
  name: {{ .Release.Name }}-service
spec:
  ports:
  - name: {{ .Values.service.portname }}
    port: {{ .Values.service.port }}
    protocol: {{ .Values.service.protocol | default "TCP" }}
    targetPort: {{ .Values.service.targetPort }}
  selector:
    app: {{ .Release.Name }}
  type: {{ .Values.service.type }}
status:
  loadBalancer: {}
```

To change the index.html we will create a configmap file as below
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-html-configmap
  namespace: default
data:
  index.html: |
    <html>
    <h1>Welcome to {{ .Values.env.name }}</h1>
    </html
```

Finally, values.yaml

```
replicaCount: 2

image:
  repository: nginx
  tag: "1.16.0"
  pullPolicy: IfNotPresent

service:
  name: nginx-service
  type: ClusterIP
  port: 80
  portname: port80
  targetPort: 9000

env:
  name: dev
```

To test if everything is ok, execute:

```
helm lint .
helm template .
helm install --dry-run my-nginx-release nginx_demo
```

To install the chart

```
helm install helm-demo nginx_demo
```

To view the helm chart
```
helm list
```

To verify kubernetes resources are created properly
```
k get deploy
k get svc
k get cm
k get po
```

To overwrite the default value file and explicitly use different values file while creating a helm chart use --values option

```
helm install helm-demo nginx-demo --values env/prod-values.yaml
```

To upgrade the chart
```
helm upgrade helm-demo nginx_demo
```

After above helm list will show new version. To rollback to previous version do
```
helm rollback helm-demo
helm rollback helm-demo 1
```

The first command will rollback to previous version and second will rollback to version number 1


To uninstall all resoruces of helm chart enter
```
helm uninstall helm-demo
```

To generate a package
```
helm package nginx_demo
```