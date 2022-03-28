#### Documentation Referred:

https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/

#### Step 1: Try connecting to Busybox pod:
```sh
kubectl exec -it busybox -- sh
```
#### Step 2: Installing nslookup utility
```sh
yum -y install bind-utils
nslookup cka-worker
```
#### Step 3: Modify the Configuration file for API Service
```sh
vi /etc/systemd/system/kube-apiserver.service
```

##### Add following flag:
```sh
--kubelet-preferred-address-types InternalIP
```
```sh
systemctl daemon-reload
systemctl restart kube-apiserver
```

#### Step 4: Verifify Connectivity
```sh
kubectl exec -it  busybox -- sh
```
