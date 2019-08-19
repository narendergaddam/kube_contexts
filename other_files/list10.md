## Kubectl Patch

```
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: patch-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: patch-demo-ctr
        image: nginx
      tolerations:
      - effect: NoSchedule
        key: dedicated
        value: test-team
```

```patch-file-containers.yaml
spec:
  template:
    spec:
      containers:
      - name: patch-demo-ctr-2
        image: redis
```
kubectl patch deployment patch-demo --patch "$(cat patch-file-containers.yaml)"


```patch-file-tolerations.yaml
spec:
  template:
    spec:
      tolerations:
      - effect: NoSchedule
        key: disktype
        value: ssd
```
```
kubectl patch deployment patch-demo --patch "$(cat patch-file-tolerations.yaml)"
```


```patch-file-2.yaml
spec:
  template:
    spec:
      containers:
      - name: patch-demo-ctr-3
        image: gcr.io/google-samples/node-hello:1.0
```
```
kubectl patch deployment patch-demo --type merge --patch "$(cat patch-file-2.yaml)"
```


## Kubectl Replication Controller (Rollback and Roll)

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx-v4
spec:
  replicas: 5
  selector:
    app: nginx
    deployment: v4
  template:
    metadata:
      labels:
        app: nginx
        deployment: v4
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.2
        args: ["nginx", "-T"]
        ports:
        - containerPort: 80
```



