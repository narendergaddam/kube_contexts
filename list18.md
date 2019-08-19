### CONTROL PLANE FAILURE

Looking at logs
For now, digging deeper into the cluster requires logging into the relevant machines. Here are the locations of the relevant log files. (note that on systemd-based systems, you may need to use journalctl instead)
```
Master
/var/log/kube-apiserver.log - API Server, responsible for serving the API
/var/log/kube-scheduler.log - Scheduler, responsible for making scheduling decisions
/var/log/kube-controller-manager.log - Controller that manages replication controllers
Worker Nodes
/var/log/kubelet.log - Kubelet, responsible for running containers on the node
/var/log/kube-proxy.log - Kube Proxy, responsible for service load balancing
Docker Services and logs

```
```
kubectl get events -n kube-system
kubectl logs [kube_scheduler_pod_name] -n kube-system
sudo systemctl status docker
sudo systemctl enable docker && systemctl start docker
sudo systemctl status kubelet
sudo systemctl enable kubelet && systemctl start kubelet
sudo su -
swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab
sudo systemctl status firewalld
sudo systemctl disable firewalld && systemctl stop firewalld
```

```
service kube-apiserver status
service kube-controller-manager status
service kube-scheduler status


journalctl -u kube-apiserver
```
### R, RB, CR, CRB
Once the API server has determined who you are (whether a pod or a user), the authorization is handled by RBAC. In this lesson, we will talk about roles, cluster roles, role bindings, and cluster role bindings.

Create a new namespace:
```
kubectl create ns web
```
The YAML for a service role:
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: web
  name: service-reader
rules:
- apiGroups: [""]
  verbs: ["get", "list"]
  resources: ["services"]
```
Create a new role from that YAML file:
```
kubectl apply -f role.yaml
```
Create a RoleBinding:
```
kubectl create rolebinding test --role=service-reader --serviceaccount=web:default -n web
```
Run a proxy for inter-cluster communications:
```
kubectl proxy
```
Try to access the services in the web namespace:
```
curl localhost:8001/api/v1/namespaces/web/services
```
Create a ClusterRole to access PersistentVolumes:
```
kubectl create clusterrole pv-reader --verb=get,list --resource=persistentvolumes
```
Create a ClusterRoleBinding for the cluster role:
```
kubectl create clusterrolebinding pv-test --clusterrole=pv-reader --serviceaccount=web:default
```
The YAML for a pod that includes a curl and proxy container:
```
apiVersion: v1
kind: Pod
metadata:
  name: curlpod
  namespace: web
spec:
  containers:
  - image: tutum/curl
    command: ["sleep", "9999999"]
    name: main
  - image: linuxacademycontent/kubectl-proxy
    name: proxy
  restartPolicy: Always
 ```
Create the pod that will allow you to curl directly from the container:
```
kubectl apply -f curl-pod.yaml
```

Get the pods in the web namespace:
```
kubectl get pods -n web
```
Open a shell to the container:
```
kubectl exec -it curlpod -n web -- sh
```
Access PersistentVolumes (cluster-level) from the pod:
```
curl localhost:8001/api/v1/persistentvolumes
```


### NETWORK POLICIES

Network policies allow you to specify which pods can talk to other pods. This helps when securing communication between pods, allowing you to identify ingress and egress rules. You can apply a network policy to a pod by using pod or namespace selectors. You can even choose a CIDR block range to apply the network policy. In this lesson, we’ll go through each of these options for network policies.

Download the canal plugin:
```
wget -O canal.yaml https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml
```
Apply the canal plugin:
```
kubectl apply -f canal.yaml
```
The YAML for a deny-all NetworkPolicy:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```
Run a deployment to test the NetworkPolicy:
```
kubectl run nginx --image=nginx --replicas=2
```
Create a service for the deployment:
```
kubectl expose deployment nginx --port=80
```
Attempt to access the service by using a busybox interactive pod:
```
kubectl run busybox --rm -it --image=busybox /bin/sh
#wget --spider --timeout=1 nginx
```

The YAML for a pod selector NetworkPolicy:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-netpolicy
spec:
  podSelector:
    matchLabels:
      app: db
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: web
    ports:
    - port: 5432
```
Label a pod to get the NetworkPolicy:

kubectl label pods [pod_name] app=db
The YAML for a namespace NetworkPolicy:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ns-netpolicy
spec:
  podSelector:
    matchLabels:
      app: db
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          tenant: web
    ports:
    - port: 5432
 ```
The YAML for an IP block NetworkPolicy:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ipblock-netpolicy
spec:
  podSelector:
    matchLabels:
      app: db
  ingress:
  - from:
    - ipBlock:
        cidr: 192.168.1.0/24
```
The YAML for an egress NetworkPolicy:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: egress-netpol
spec:
  podSelector:
    matchLabels:
      app: web
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: db
    ports:
    - port: 5432
```
