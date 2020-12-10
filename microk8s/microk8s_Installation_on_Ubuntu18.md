# Install microk8s:

## Download and install microk8s:

Refer: https://microk8s.io/

```shell
#sudo apt-get -y update
#sudo apt-get -y upgrade
#sudo snap install microk8s --classic
#sudo microk8s status --wait-ready
#sudo microk8s enable dashboard dns
```

 

## Create alias for "microk8s kubectl" and default editor:

```shell
#nano .bashrc
alias mkctl="microk8s kubectl"
export EDITOR=nano
export VISUAL=nano

#source .bashrc
```

 

## Modify `sudoers` file to execute command without `sudo`: useful for creating CI:CD pipeline:

From administrator user name="anadmin", switch to root user and modify `sudoers` file

```shell
#sudo su -
#echo "anadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
#exit
#sudo usermod -a -G microk8s anadmin
#sudo chown -f -R anadmin ~/.kube
#exit
```

### Check Minikube status:

```shell
#mkctl version
#mkctl cluster-info
#mkctl get pods --all-namespaces
```

 

# Config kubernetes-dashboard:

Reference URL: https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/



## Edit kubernetes-dashboard service:

```shell
#mkctl edit service kubernetes-dashboard -n kube-system
Before:
 type: ClusterIP
After:
 type: NodePort
```

 

## Check kubernetes-dashboard service:

```
#mkctl get service kubernetes-dashboard -n kube-system
NAME          TYPE    CLUSTER-IP   EXTERNAL-IP  PORT(S)     AGE
kubernetes-dashboard  NodePort  10.109.162.67  <none>    443:30000/TCP  87m

#ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:ae:e9:85 brd ff:ff:ff:ff:ff:ff
    inet 10.0.5.58/16 brd 10.0.255.255 scope global dynamic enp0s3
       valid_lft 82006sec preferred_lft 82006sec
    inet6 fe80::a00:27ff:feae:e985/64 scope link
       valid_lft forever preferred_lft forever
```

 In this sample, `HostPort`=30000 and `HostIP`=10.0.5.58.



## Creating a Sample Service Account:

Reference URL: https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

```yaml
#nano mk-dashboard-adminuser.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system

#mkctl apply -f mk-dashboard-adminuser.yaml
```




## Creating a ClusterRoleBinding:

In most cases after provisioning cluster using kops, kubeadm or any other popular tool, the ClusterRole cluster-admin already exists in the cluster. We can use it and create only ClusterRoleBinding for our ServiceAccount. If it does not exist then you need to create this role first and grant required privileges manually.

```yaml
#nano mk-dashboard-add-adminuser-role.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system

#mkctl apply -f mk-dashboard-add-adminuser-role.yml
```

 

## Getting a Bearer Token:

```shell
#mkctl -n kube-system describe secret $(mkctl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

 

## Access to Dashboard using WebBrowser:

- https://HostIP:HostPort
- Accept Self-signed SSL security issue warning of web browser.
- Choose login using `Token` option and copy&paste BearerToken in `Enter Token` box



## Clean up and next steps:

Remove the admin `ServiceAccount` and `ClusterRoleBinding`

```shell
#mkctl -n kube-system delete serviceaccount admin-user
#mkctl -n kube-system delete clusterrolebinding admin-user
```



## Start and stop Kubernetes to save battery:

```shell
#microk8s start
#microk8s stop
```

