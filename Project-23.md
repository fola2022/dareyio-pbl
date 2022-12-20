# PERSISTING DATA IN KUBERNETES
### Setting up EKS cluster with a single commandline:
```
eksctl create cluster \
  --name my-eks-clusters \
  --version 1.21 \
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

