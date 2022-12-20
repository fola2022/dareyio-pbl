# PERSISTING DATA IN KUBERNETES
## INTRODUCTION
### The pods created in Kubernetes are ephemeral, they don't run for long. When a pod dies, any data that is not part of the container image will be lost when the container is restarted because Kubernetes is best at managing stateless applications which means it does not manage data persistence. To ensure data persistent, the PersistentVolume PV, PersistentVolumeClaim PVC or Configmap resource can be implement to acheive this.
### STEP 1: Setting Up AWS Elastic Kubernetes Service With EKSCTL
#### Installing eksctl
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
#### Setting up EKS cluster with a single commandline:
```
eksctl create cluster \
  --name my-eks-clusters \
  --version 1.24 \
  --region us-east-1 \
  --nodegroup-name worker-nodes \
  --node-type t2.medium \
  --nodes 2
```
<img width="520" alt="eks cluster created" src="https://user-images.githubusercontent.com/112771723/208666146-5384d833-fbfd-4f07-9fd6-6b30fc4edcdd.png">

### STEP 2: Creating Persistent Volume Manually For The Nginx Application
#### Creating a deployment manifest file for the Nginx application and applying it:
#### nginx-pod.yaml manifest file
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```        
<img width="233" alt="nginx-pod" src="https://user-images.githubusercontent.com/112771723/208665974-faab1f13-eb0c-44a9-bed3-2b5d63eac0b0.png">
<img width="335" alt="nginx-pod created" src="https://user-images.githubusercontent.com/112771723/208666012-f037aac2-0e2f-4136-8855-484c024db1e2.png">

###Verifying that the pod is running: $ kubectl get pod
Exec into the pod and navigating to the nginx configuration file:
