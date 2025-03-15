# Create user with read only on K8s Cluster

## 1 ) Generate certificate
```
openssl genrsa -out read-only-user.key 2048
openssl req -new -key read-only-user.key -out read-only-user.csr -subj "/CN=read-only-user"
openssl x509 -req -in read-only-user.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out read-only-user.crt -days 360
openssl x509 -req -in read-only-user.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out read-only-user.crt -days 360
```
## 2 ) Create role and roleBinding
```
kubectl apply -f read-only-role.yaml
kubectl apply -f read-only-rolebinding.yaml
```
## 3 ) Build Kube config
```
kubectl config set-cluster kubernetes --certificate-authority=/etc/kubernetes/pki/ca.crt --server=https://kubernetes-API:6443 --kubeconfig=read-only-user.kubeconfig
kubectl config set-credentials read-only-user --client-certificate=read-only-user.crt --client-key=read-only-user.key --kubeconfig=read-only-user.kubeconfig
kubectl config set-context read-only-user-context --cluster=kubernetes --namespace=default --user=read-only-user --kubeconfig=read-only-user.kubeconfig
kubectl config use-context read-only-user-context --kubeconfig=read-only-user.kubeconfig
```
## 4 ) change path by config
```
certificate-authority --> certificate-authority-data => content from : /root/.kube/config

client-certificate --> client-certificate-data  => content from : base64 -w 0 read-only-user.crt

client-key --> client-key-data => content from : base64 -w 0 read-only-user.key
```
## 5 ) Test
```
kubectl --kubeconfig read-only-user.kubeconfig logs nginx
```
