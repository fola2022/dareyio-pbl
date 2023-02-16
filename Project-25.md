##  DEPLOYING AND PACKAGING APPLICATIONS INTO KUBERNETES WITH HELM
### Deploying Artifactory into kubernetes using Artifactory helm chart in artifacthub
#### First, created a namespace called tools. The Artifactory would be install in the namespace
```
kubectl create namespace tools
```
#### Installing the artifactory helm chart
```
helm repo add jfrog https://charts.jfrog.io
helm repo update
helm upgrade --install artifactory jfrog/artifactory --version 107.38.10 -n tools
```
<img width="792" alt="Artifactory installed" src="https://user-images.githubusercontent.com/112771723/219381820-b65ede9e-7376-406b-aa7c-8722a525da4b.png">
#### Checking the installation
```
kubectl get pod -n tools
kubectl get svc -n tools
```
<img width="947" alt="artifactory pod and svc running" src="https://user-images.githubusercontent.com/112771723/219382396-979c0f72-2e97-4261-83af-db728e36b3dd.png">

#### Reaching the artifactory on broswer using the nginx proxy service, which is the loadbalancer
```
 kubectl get svc artifactory-artifactory-nginx -n tools
 ```
#### The load balancer URL is copied and paste on the broswer
<img width="805" alt="artifactory broswer" src="https://user-images.githubusercontent.com/112771723/219383496-39660518-941e-4f2a-a2e1-69005fa768a8.png">

### DEPLOYING INGRESS CONTROLLER AND MANAGING INGRESS RESOURCES
#### Using the Helm approach, according to the official guide;
```
helm upgrade --install ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--namespace ingress-nginx --create-namespace
```
<img width="765" alt="installed ingress controller" src="https://user-images.githubusercontent.com/112771723/219385084-95515354-630c-45d6-a024-27c2f813a568.png">

#### Checking the installation
```
kubectl get pods --namespace=ingress-nginx
kubectl get svc -n ingress-nginx
kubectl get ingressclass -n ingress-nginx
```
<img width="948" alt="ingress controller use" src="https://user-images.githubusercontent.com/112771723/219386625-66ec0e76-f96b-4180-baf5-017288c6f78f.png">

#### The ingress controller load balancer on AWS
<img width="724" alt="loadblancer" src="https://user-images.githubusercontent.com/112771723/219385801-2c63bd82-99be-4026-80e1-4f1a37be049b.png">

### Deploy Artifactory Ingress
#### Now, it is time to configure the ingress so that it can route traffic to the Artifactory internal service, through the ingress controller’s load balancer.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: artifactory
spec:
  ingressClassName: nginx
  rules:
  - host: "tooling.artifactory.yellowgem.tk"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: artifactory
            port:
              number: 8082
```  
<img width="327" alt="ingress yml" src="https://user-images.githubusercontent.com/112771723/219388004-6de5895c-6224-409b-b5f5-a3954c09a2ca.png">

```
kubectl apply -f ingress.yaml -n tools
kubectl get ingress -n tools
```
<img width="869" alt="ingress created use" src="https://user-images.githubusercontent.com/112771723/219389518-083e2cc3-c15e-44e2-ad1c-6a648a0a399e.png">

### Creating Route53 record using Alias method
<img width="838" alt="record use" src="https://user-images.githubusercontent.com/112771723/219390513-61ec1c90-2da2-4b22-9a78-991cd4592efe.png">

#### Visiting the application from the browser
#### So far, we now have an application running in Kubernetes that is also accessible externally. That means if we navigate to https://tooling.artifactory.yellowgem.tk/, it should load up the artifactory application.
<img width="851" alt="ssl " src="https://user-images.githubusercontent.com/112771723/219391911-c6dbb9ce-0d09-4fb7-a073-cdcfb900e343.png">

#### The site is reachable but not secure. It is insecure because it either does not have a trusted TLS/SSL certificate, or it doesn’t have any at all.
### Making the site secure
#### - Login into Artifactory with the default username and password
#### - Activate the Artifactory License. You will need to purchase a license to use Artifactory enterprise features.
<img width="839" alt="jfrog setting" src="https://user-images.githubusercontent.com/112771723/219393362-f224b849-227d-4d3c-b24e-d4d1c066396e.png">

### DEPLOYING CERT-MANAGER AND MANAGING TLS/SSL FOR INGRESS
#### Installing cert-manager using helm chat
```
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager --namespace cert-manager --version v1.11.0 jetstack/cert-manager
```
<img width="673" alt="cert-manager installed" src="https://user-images.githubusercontent.com/112771723/219394905-fde53c92-5ef5-42be-a7cd-b394890ae291.png">

#### Creating a Certificate Issuer
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  namespace: "cert-manager"
  name: "letsencrypt-prod"
spec:
  acme:
    server: "https://acme-v02.api.letsencrypt.org/directory"
    email: "infradev@oldcowboyshop.com"
    privateKeySecretRef:
      name: "letsencrypt-prod"
    solvers:
    - selector:
        dnsZones:
          - "yellowgem.tk"
      dns01:
        route53:
          region: "us-east-1"
          hostedZoneID: "Z2CD4NTR2FDPZ"
``` 
<img width="340" alt="clusterissuer" src="https://user-images.githubusercontent.com/112771723/219396857-48eccf95-7d35-4457-8139-6f29e3b7d007.png">

```
kubectl apply -f clusterissuer.yaml
```
<img width="314" alt="clusterissuer created" src="https://user-images.githubusercontent.com/112771723/219396668-75ba3b94-5015-4d02-9c36-ac756392fe1d.png">

#### To ensure that every created ingress also has TLS configured, we will need to update the ingress manifest with TLS specific configurations.
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    kubernetes.io/ingress.class: nginx
  name: artifactory
spec:
  rules:
  - host: "tooling.artifactory.yellowgem.tk"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: artifactory
            port:
              number: 8082
  tls:
  - hosts:
    - "tooling.artifactory.yellowgem.tk"
    secretName: "tooling.artifactory.yellowgem.tk"
```
#### Running the folloing command to see each resources at each phase
```
kubectl get certificaaterequest 
kubectl get order
kubectl get challenge
kubectl get certificate
```
<img width="729" alt="cert  customerresource" src="https://user-images.githubusercontent.com/112771723/219398540-61845a3b-0962-450d-a190-2c03c386c912.png">



