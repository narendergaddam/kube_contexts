## Ingress

Layer 7 Load Balancer - Build in Kubernetes

### INGRESS CONTROLLER

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    run: nginx-ingress-controller
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx-ingress-controller
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: nginx-ingress-controller
    spec:
      containers:
      - image: nginx-ingress-controller:0.21.0
        name: nginx-ingress-controller
        resources: {}
      args:
      - /nginx-ingress-controller
      - --configmap=$(POD_NAMESPACE)/nginx-configuration
      env:
      - name: POD_NAME
        valueFrom:
           fieldRef:
              fieldPath: metadata.name
      - name: POD_NAMESPACE
        valueFrom:
           fieldRef:
              fieldPath: metadata.namespace
      ports:
      - name: http
        containerPort: 80
      - name: https
        containerPort: 443
              
        
status: {}
```

```
kind: ConfigMap
apiVersion: v1
metadata:
  name:  nginx-configuration
```

```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nginx-ingress
  name: nginx-ingress
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  - name: https
    port: 443
    protocol: TCP
    targetPort: 443
  selector:
    app: nginx-ingress
  type: NodePort
```

`Rules, cluster roles and role bingings`
```
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: null
  name: nginx-ingress-serviecaccount
```


### Ingress Rules


```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - host: wear.test.com
    http:
      paths:
      - backend:
           serviceName: wear-services
           servicePort: 80
  - host: test.dubble.com
    http:
      paths:
      - backend:
           serviceName: watch-nde
           servicePort: 80
```