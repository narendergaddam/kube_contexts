### TLS


### CA 

```
Generate Private Keys
openssl genrsa -out ca.key 2048

Certificate Signing Request 
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr

Sign Certificates
openssl x509 -req -in ca.csr -signkey ca.key -out=ca.crt
```


### Client Certificates
#### ADMIN USER , SCHEDULER, CONTROLLER, KUBE-PRXY, APISERVER, KUBELET
```
Generate Private Keys
openssl genrsa -out admin.key 2048

Certificate Signing Request 
openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr

Sign Certificates
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
``` 



### Server Certificates
#### ETCD, KUBE-API, 
