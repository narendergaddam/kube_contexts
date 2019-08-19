## Connect Fron End to Backend using Service

Deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 7
  selector:
    matchLabels:
      app: hello
      tier: backend
      track: stable
  template:
    metadata:
      labels:
        app: hello
        tier: backend
        track: stable
    spec:
      containers:
      -  name: hello
         image: "gcr.io/google-samples/hello-go-gke:1.0"
         ports:
            - name: http
              containerPort: 80
```

```
kind: Service
apiVersion: v1
metadata:
  name: hello
spec:
  selector:
    app: hello
    tier: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: http
```

    