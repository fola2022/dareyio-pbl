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

#### Since the type of service created for the Nginx pod is a ClusterIP which cannot be accessed externally, doing port-forwarding in order to bind the machine's port to the ClusterIP service port, i.e, tunnelling traffic through the machine's port number to the port number of the nginx-service: 
```
kubectl port-forward svc/nginx-service 8089:80
```
<img width="463" alt="port" src="https://user-images.githubusercontent.com/112771723/208485902-1edad68f-640c-4c84-9dfc-338acccb263b.png">
<img width="616" alt="nginx on broswer" src="https://user-images.githubusercontent.com/112771723/208485980-1c062b8f-c0e1-40b9-bad0-08b70ffd0cc2.png">

### STEP 3: Creating A Replica Set
#### The replicaSet object helps to maintain a stable set of Pod replicas running at any given time to achieve availability in case one or two pods dies.
#### Deleting the nginx-pod: `kubectl delete pod nginx-pod`
#### Creating the replicaSet manifest file and applying it: `kubectl apply -f rs.yaml`
#### rs.yaml manifest file
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    app: nginx-pod
  template:
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
<img width="200" alt="replicaset" src="https://user-images.githubusercontent.com/112771723/208486712-52480ab1-e086-4488-81a5-64de70233690.png">
<img width="328" alt="rs created" src="https://user-images.githubusercontent.com/112771723/208486629-4e520653-754c-40d4-9d5a-4c4658bb6b0d.png">

```
kubectl describe nginx-rs
kubectl get pod
```
<img width="323" alt="3 replica" src="https://user-images.githubusercontent.com/112771723/208486894-9ec15df0-f2d2-4911-941b-3ccb553294cc.png">
<img width="503" alt="describe nginx rs" src="https://user-images.githubusercontent.com/112771723/208489968-ae1847c8-3685-4129-86cf-2699ec4537e0.png">

#### Deleting one of the pods will cause another one to be scheduled and set to run: kubectl delete pod nginx-rs-gprcc
#### Another pod scheduled
<img width="323" alt="del cre" src="https://user-images.githubusercontent.com/112771723/208487066-289eb41a-b3d5-4710-9cd4-1d8dada35a95.png">

#### Two ways pods can be scaled are Imperative and Declarative
Imperative method is by running a command on the CLI:
```
kubectl scale --replicas 5 replicaset nginx-rs
```
<img width="359" alt="imperative scale rs" src="https://user-images.githubusercontent.com/112771723/208487593-50ef8a5f-8af8-4861-bc4a-b7de7e6fd827.png">

#### Declarative method is done by editing the rs.yaml manifest and changing to the desired number of replicas and applying the update
#### In some cases, we want ReplicaSet to manage our existing containers that match certain criteria, we can use the same simple label matching or we can use some more complex conditions, such as:
```
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      env: prod
    matchExpressions:
    - { key: tier, operator: In, values: [frontend] }
  template:
    metadata:
      name: nginx
      labels: 
        env: prod
        tier: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
          protocol: TCP
```          
#### nginx-service.yaml manifest file
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    tier: frontend
  ports:
    - protocol: TCP
      port: 80 # This is the port the Loadbalancer is listening at
      targetPort: 80 # This is the port the container is listening at
```  
#### After applying the configuration
```
kubectl apply -f nginx-service.yaml
```
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
<img width="708" alt="nginx loadbalancer" src="https://user-images.githubusercontent.com/112771723/208490390-51579068-63b8-4601-bddb-a2ab609c149d.png">

---
kubectl get service nginx-service -o yaml
```
<img width="943" alt="lb" src="https://user-images.githubusercontent.com/112771723/208490153-28a0c501-dc6f-480c-a885-b1642af59d72.png">

#### STEP 4: Creating Deployment
A Deployment is another layer above ReplicaSets and Pods, It manages the deployment of ReplicaSets and allows for easy updating of a ReplicaSet as well as the ability to roll back to a previous version of deployment. To see it in action:

Deleting the ReplicaSet that was created before: $ kubectl delete rs nginx-rs
Creating deployment manifest file called deployment.yaml and applying it:$ kubectl apply -f deployment.yaml

