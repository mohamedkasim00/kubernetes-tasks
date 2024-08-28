# Kubernetes Tasks with Questions, Answers

This repository contains Kubernetes YAML files, commands, and detailed explanations for various tasks.

## 1. How many DaemonSets are created in the cluster in all namespaces?
- **Command:**
  ```bash
  kubectl get daemonsets --all-namespaces
#This command lists all DaemonSets across all namespaces in the cluster.

#2. What DaemonSets exist in the kube-system namespace?
```bash
kubectl get daemonsets -n kube-system
#This command lists all DaemonSets in the kube-system namespace.

#3. What is the image used by the Pod deployed by the kube-proxy DaemonSet?
```bash
kubectl get daemonset kube-proxy -n kube-system -o=jsonpath='{.spec.template.spec.containers[0].image}'
#This command retrieves the image used by the container in the kube-proxy DaemonSet, which is crucial for understanding the version of the proxy running in the cluster.

#4. Deploy a DaemonSet for FluentD Logging
Specifications:
Name: elasticsearch
Namespace: kube-system
Image: k8s.gcr.io/fluentd-elasticsearch:1.20
YAML Configuration
```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      containers:
      - name: fluentd-elasticsearch
        image: k8s.gcr.io/fluentd-elasticsearch:1.20
        resources:
          limits:
            memory: 200Mi
            cpu: 100m
          requests:
            memory: 200Mi
            cpu: 100m

#This YAML configuration creates a DaemonSet named elasticsearch in the kube-system namespace using the specified FluentD image. DaemonSets ensure that a copy of the pod runs on all or some nodes.


#5-Deploy a Pod named nginx-pod using the nginx:alpine image with the label tier=backend
YAML Configuration
```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    tier: backend
spec:
  containers:
  - name: nginx-container
    image: nginx:alpine
#This configuration creates a Pod named nginx-pod using the nginx:alpine image with a label tier=backend. The label is crucial for service selection.

#6- Deploy a test Pod using the nginx:alpine image
YAML Configuration
```bash
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: test-container
    image: nginx:alpine
#his deploys a simple test pod using the nginx:alpine image

#7. Create a Service backend-service to expose the backend application within the cluster on port 80
YAML Configuration
```bash
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    tier: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
#This service exposes the nginx-pod (which has the label tier=backend) on port 80 within the cluster.

#8. Try to curl the backend-service from the test-pod. What is the response?
```bash
kubectl exec -it test-pod -- curl backend-service
#You should receive the default nginx welcome page HTML if everything is set up correctly.
#Troubleshooting: If the connection fails, verify the pod labels, service configuration, and pod status.

#9. Deploy a Deployment named web-app using the nginx image with 2 replicas
YAML Configuration
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx-container
        image: nginx
#This creates a deployment named web-app with 2 replicas, each running the nginx image.

#10. Expose the web-app Deployment as a Service web-app-service on port 80 and NodePort 30082
```bash
kubectl expose deployment web-app --type=NodePort --port=80 --target-port=80 --name=web-app-service
#Edit Service to Specify NodePort
```bash
kubectl edit svc web-app-service
#Add the following under ports:
```bash
nodePort: 30082

#11. Access the Web App from the Node
```bash
http://<node-ip>:30082
# Replace <node-ip> with the actual IP of one of your nodes to access the web app.

#12. How many static pods exist in this cluster in all namespaces?
```bash
kubectl get pods --all-namespaces --field-selector=status.phase=Running
#static pods are managed directly by the kubelet on each node and aren’t part of a Deployment or DaemonSet. You can identify them by their unique names.

#13. On which nodes are the static pods created currently?
```bash
kubectl get pods --all-namespaces -o wide
#This command lists all running pods across all namespaces, along with the node they are running on. Static pods are usually associated with system components and run on master nodes.


#Repository Structure
- **nginx-pod.yaml**: Deploys a Pod using the `nginx:alpine` image.
- **test-pod.yaml**: Deploys a test Pod using the `nginx:alpine` image.
- **backend-service.yaml**: Exposes the backend application within the cluster on port 80.
- **elasticsearch-daemonset.yaml**: Deploys a DaemonSet for FluentD logging.
- **web-app-deployment.yaml**: Deploys a web application using the `nginx` image with 2 replicas.

## How to Apply

To apply these configurations, use the following command:

```bash
kubectl apply -f <filename>.yaml
