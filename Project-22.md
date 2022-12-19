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
