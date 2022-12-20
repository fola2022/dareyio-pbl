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

#### Verifying that the pod is running:
```
kubectl get deploy
kubectl get pod
```
<img width="465" alt="nginx pod created and running" src="https://user-images.githubusercontent.com/112771723/208768526-6b2c86e0-852a-4c2b-923b-6f546ab29510.png">

#### Exec into the pod and navigating to the nginx configuration file:
<img width="764" alt="exec in pod" src="https://user-images.githubusercontent.com/112771723/208768916-f0ff8cca-9c2c-44e7-bc67-85ec18816c03.png">

#### When creating a volume it must exists in the same region and availability zone as the EC2 instance running the pod. To confirm which node is running the pod:
```
kubectl get po nginx-deployment-5d6cf97577-96h1f -o wide
```
#### To check the Availability Zone where the node is running:kubectl describe node ip-192-168.25.85.us-west-1.compute.internal
<img width="929" alt="describe node az" src="https://user-images.githubusercontent.com/112771723/208770343-bf9caadf-0a99-4d1b-bb1d-a1c16429a05e.png">

#### Creating a volume in the Elastic Block Storage section in AWS in the same AZ as the node running the nginx pod which will be used to mount volume into the Nginx pod.
<img width="751" alt="volume created" src="https://user-images.githubusercontent.com/112771723/208770532-267ae9f5-df9f-4a8a-8691-f87af5e55ad8.png">

#### Updating the deployment configuration with the volume spec and volume mount:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 1
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
        volumeMounts:
        - name: nginx-volume
          mountPath: /usr/share/nginx/
      volumes:
      - name: nginx-volume
        awsElasticBlockStore:
          volumeID: "vol-00dad625ccc8aba34"
          fsType: ext4
```
<img width="290" alt="volume added" src="https://user-images.githubusercontent.com/112771723/208770981-f010edde-8740-486f-b345-9e401412bd5d.png">
<img width="467" alt="pod creating after adding volume" src="https://user-images.githubusercontent.com/112771723/208771183-c3c73c3f-9086-4c28-8d8e-6a9457ec5400.png">

#### But the problem with this configuration is that when we port forward the service and try to reach the endpoint, we will get a 403 error. This is because mounting a volume on a filesystem that already contains data will automatically erase all the existing data. To solve this issue is by implementing Persistent Volume(PV) and Persistent Volume claims(PVCs) resource.
### STEP 3: Managing Volumes Dynamically With PV and PVCs
#### PVs are resources in the cluster. PVCs are requests for those resources and also act as claim checks to the resource. By default in EKS, there is a default storageClass configured as part of EKS installation which allow us to dynamically create a PV which will create a volume that a Pod will use.
#### Verifying that there is a storageClass in the cluster: `kubectl get storageclass`
<img width="630" alt="storage class" src="https://user-images.githubusercontent.com/112771723/208771822-bd6dba5b-45bf-4815-9f39-f609340f2504.png">

#### Creating a manifest file for a PVC, and based on the gp2 storageClass a PV will be dynamically created:
```
apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: nginx-volume-claim
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 2Gi
      storageClassName: gp2
 ```     
 <img width="196" alt="pvc yaml" src="https://user-images.githubusercontent.com/112771723/208771983-3ff4754f-ba1e-4711-bc29-99dc57936fd1.png">
 
 #### Checking the setup: `kubectl get pvc`
 <img width="445" alt="pvc get pending" src="https://user-images.githubusercontent.com/112771723/208772266-a5b57612-8c58-444a-9608-f7777ada687c.png">
 
#### Checking for the volume binding section: `kubectl describe pvc`
<img width="811" alt="describe pvc" src="https://user-images.githubusercontent.com/112771723/208772536-74cc5574-b439-4936-b906-7ac730a232f1.png">

#### The PVC created is in pending state because PV is not created yet. Editing the nginx-pod.yaml file to create the PV:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 1
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
        volumeMounts:
        - name: nginx-volume-claim
          mountPath: /tmp/folah
      volumes:
      - name: nginx-volume-claim
        persistentVolumeClaim:
          claimName: nginx-volume-claim
```
<img width="287" alt="pvc nginx pod volume" src="https://user-images.githubusercontent.com/112771723/208772967-9c4f3605-4aeb-457a-b4c4-2b02e5ad7793.png">

#### The '/tmp/folah' directory will be persisted, and any data written in there will be stored permanetly on the volume, which can be used by another Pod if the current one gets replaced.
### STEP 4: Use Of ConfigMap As A Persistent Storage
#### ConfigMap is an API object used to store non-confidential data in key-value pairs. It is a way to manage configuration files and ensure they are not lost as a result of Pod replacement.
#### To demonstrate this, the HTML file that came with Nginx will be used.
#### Exec into the container and copying the HTML file:
```
kubectl exec -it nginx-deployment-5d6cf97577-96hlf bash
cat /usr/share/nginx/html/index.html 
```
<img width="746" alt="exec and cat html" src="https://user-images.githubusercontent.com/112771723/208774249-56113844-3ada-4460-ba90-1e0d269ff955.png">

#### Creating the ConfigMap manifest file and customizing the HTML file and applying the change:
#### nginx-configmap.yaml file
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: website-index-file
data:
  # file to be mounted inside a volume
  index-file: |
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to Nginx!</title>
    <style>
    html { color-scheme: light dark; }
    body { width: 35em; margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
    </head>
    <body>
    <h1>Welcome to Nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
```
<img width="356" alt="configmap yaml created" src="https://user-images.githubusercontent.com/112771723/208775021-7c60ed9a-9e9b-43dd-938a-1cf05ff70c25.png">

#### Updating the deployment file to use the configmap in the volumeMounts section
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 1
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
        volumeMounts:
          - name: config
            mountPath: /usr/share/nginx/html
            readOnly: true
      volumes:
      - name: config
        configMap:
          name: website-index-file
          items:
          - key: index-file
            path: index.html
```
<img width="310" alt="nginx pod updated configmap" src="https://user-images.githubusercontent.com/112771723/208775285-f57b8bdf-633c-41b0-9dc8-027f46a98519.png">

#### Now the index.html file is no longer ephemeral because it is using a configMap that has been mounted onto the filesystem. This is now evident when exec into the pod and list the /usr/share/nginx/html directory
#### To see the configmap created: `kubectl get configmap`
#### To see the change in effect, updating the configmap manifest: `kubectl edit cm website-index-file`
```
 <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to ChassTech Services!</title>
    <style>
    html { color-scheme: light dark; }
    body { width: 35em; margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
    </head>
    <body>
    <h1>Welcome to DAREY.IO!</h1>
    <p>If you see this page, It means you have successfully updated the configMap data Kubernete.</p>

    <p>For online documentation and support please refer to
    <a href="http://DAREY.IO/">DAREY.IO</a>.<br/>
    Commercial support is available at
    <a href="http://DAREY.IO/">DAREY.IO</a>.</p>

    <p><em>Thank you and make sure you are on Darey's Masterclass Program.</em></p>
    </body>
    </html>
```    
<img width="755" alt="config edited" src="https://user-images.githubusercontent.com/112771723/208776990-0cf6e703-1d16-4179-800a-a3a9d5586123.png">
<img width="569" alt="port configmap" src="https://user-images.githubusercontent.com/112771723/208775699-66522237-9953-49d0-a209-cbe150b7d5af.png">
<img width="689" alt="broswer end" src="https://user-images.githubusercontent.com/112771723/208777147-96cda622-2547-4736-91e0-7d3fda1fcfb4.png">

            
