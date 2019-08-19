### Pull an Image from a Private Registry


```
docker login
cat ~/.docker/config.json
```
- Create the secret based on existing docker credentials

```
λ ~/ kubectl create secret generic regcred \
    --from-file=.dockerconfigjson=/Users/malipr/.docker/config.json --type=kubernetes.io/dockerconfigjson

kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>

```

- Create the pod that uses your secret 
```
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred
```
