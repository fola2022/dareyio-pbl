# DEPLOYING APPLICATIONS INTO KUBERNETES CLUSTER
## INTRODUCTION
### This project demonstrates how containerised applications are deployed as pods in Kubernetes and how to access the application from the browser.
### STEP 1: Creating A Pod For The Nginx Application
#### nginx-pod.yaml manifest file
```
apiVersion: v1
kind: Pod
metadata:
name: nginx-pod
spec:
containers:
- image: nginx:latest
name: nginx-pod
ports:
- containerPort: 80
  protocol: TCP
```
#### Creating nginx pod by applying the manifest file:
```
kubectl apply -f nginx-pod.yaml
```
#### Running the following commands to inspect the setup:
```
kubectl get pod nginx-pod --show-labels
kubectl get pod nginx-pod -o wide
kubectl describe pod nginx-pod
```
<img width="766" alt="nginx pod" src="https://user-images.githubusercontent.com/112771723/208479598-41647079-8b6c-4307-b025-0110fa5caf13.png">

### STEP 2: Accessing The Nginx Application Through A Browser
#### First of all, Trying to access the Nginx Pod through its IP address from within the Kubernetes cluster. To do this an image that already has curl software installed is needed.
#### Running the kubectl command to run the container that has curl software in it as a pod
```
kubectl run curl --image=dareyregistry/curl -i --tty
```
#### Running curl command and pointing it to the IP address of the Nginx Pod: `curl -v 192.168.11.95:80`
<img width="536" alt="curl" src="https://user-images.githubusercontent.com/112771723/208481018-5c77dc1e-bfeb-425c-9ca9-f8ce475e9fc4.png">

#### Creating a service for the Nginx pod.
#### Creating service for the nginx pod by applying the manifest file: `kubectl apply -f nginx-service.yaml`
#### First, nginx-pod.yaml was edited by adding labels
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx-pod  
spec:
  containers:
  - image: nginx:latest
    name: nginx-pod
    ports:
    - containerPort: 80
      protocol: TCP
```  
<img width="179" alt="nginx pod edited" src="https://user-images.githubusercontent.com/112771723/208483507-f1832ffa-4533-4eae-8dae-94d43f144202.png">

#### nginx-service.yaml manifest file
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-pod 
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
<img width="196" alt="nginxxx" src="https://user-images.githubusercontent.com/112771723/208483410-c99a4071-49cb-41fa-9830-c22631f7ea97.png">
<img width="438" alt="nginx service" src="https://user-images.githubusercontent.com/112771723/208483108-bf7fef09-78a4-4973-9ca8-af573e0e28b2.png">

#### Since the type of service created for the Nginx pod is a ClusterIP which cannot be accessed externally, we can do port-forwarding in order to bind the machine's port to the ClusterIP service port, i.e, tunnelling traffic through the machine's port number to the port number of the nginx-service: $ kubectl port-forward svc/nginx-service 8089:80
deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tooling-deployment
  labels:
    app: tooling-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tooling-app
  template:
    metadata:
      labels:
        app: tooling-app
    spec:
      containers:
      - name: tooling
        image: folah/tooling:0.0.2
        ports:
        - containerPort: 80
```
