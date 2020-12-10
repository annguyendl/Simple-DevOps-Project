# Install Docker CE:

Reference URL : https://docs.docker.com/install/linux/docker-ce/centos/#install-using-the-repository

 

## Install the required packages:

```shell
#sudo apt-get update -y
#sudo apt-get upgrade -y
```

 

Install the latest version of Docker CE:

```shell
#sudo apt install docker.io
```

 

Enable and start Docker service:

```shell
#sudo systemctl enable docker
#sudo systemctl start docker
#sudo systemctl status docker
#sudo docker version
```

 

# Install kubectl:

## Get latest version of kubectl:

```shell
#KUBE_VERSION=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)
#echo $KUBE_VERSION
#cd /opt/
#sudo curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/${KUBE_VERSION}/bin/linux/amd64/kubectl
#sudo chmod +x kubectl
#sudo mv -f kubectl /usr/local/bin/
#kubectl version
```

 It will show error here because no kubenetes' cluster has configured.



# Install Minikube:

## Install conntrack-tools required by minukube:

```shell
#sudo apt install conntrack
```

 

## Download and install minikube:

```shell
#cd ~
#sudo curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
#sudo cp minikube-linux-amd64 /usr/local/bin/minikube
#sudo chmod 755 /usr/local/bin/minikube
#minikube version
```

 

## Start Minikube:

```shell
#minikube start --vm-driver=none --alsologtostderr -v=8
! The 'none' driver is designed for experts who need to integrate with an existing VM
\* Most users should use the newer 'docker' driver instead, which does not require root!
\* For more information, see: https://minikube.sigs.k8s.io/docs/reference/drivers/none/
*
! kubectl and minikube configuration will be stored in /root
! To use kubectl or minikube commands as your own user, you may need to relocate them. For example, to overwrite your own settings, run:
*
 \- sudo mv /root/.kube /root/.minikube $HOME
 \- sudo chown -R $USER $HOME/.kube $HOME/.minikube
*
\* This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
\* Verifying Kubernetes components...
\* Enabled addons: default-storageclass, storage-provisioner
\* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

 

### Check Minikube status:

```shell
#kubectl version
#kubectl cluster-info
#kubectl get pods --all-namespaces
```

 

#### In-case we have some errors with pods:

(*1) coredns error: CrashLoopBackOff: dial tcp ip:443: connect: no route to host

```shell
#sudo iptables -P FORWARD ACCEPT
#sudo iptables --flush
#sudo iptables -tnat --flush
#sudo minikube stop
#sudo minikube start
```

 

# Install kubernetes-dashboard:

Reference URL: https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

## Download latest recommend deployment:

```shell
#curl -LO https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
#kubectl apply -f ./recommended.yaml
```

 

## Edit kubernetes-dashboard service:

```shell
#kubectl edit service kubernetes-dashboard -n kubernetes-dashboard
Before:
 type: ClusterIP
After:
 type: NodePort
```

 

## Check kubernetes-dashboard service:

```
#kubectl get service kubernetes-dashboard -n kubernetes-dashboard
NAME          TYPE    CLUSTER-IP   EXTERNAL-IP  PORT(S)     AGE
kubernetes-dashboard  NodePort  10.109.162.67  <none>    443:30000/TCP  87m
```

 

## Creating a Sample Service Account:

Reference URL: https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

```yaml
#nano dashboard-adminuser.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

#kubectl apply -f dashboard-adminuser.yaml
```




## Creating a ClusterRoleBinding:

In most cases after provisioning cluster using kops, kubeadm or any other popular tool, the ClusterRole cluster-admin already exists in the cluster. We can use it and create only ClusterRoleBinding for our ServiceAccount. If it does not exist then you need to create this role first and grant required privileges manually.

```yaml
#nano dashboard-add-adminuser-role.yml
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
  namespace: kubernetes-dashboard

#kubectl apply -f dashboard-add-adminuser-role.yml
```

 

## Getting a Bearer Token:

```
#kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

 

## Access to Dashboard using WebBrowser:

- HostIP:PublicPort
- Choose login using `Token` option and copy&paste BearerToken in `Enter Token` box



## Clean up and next steps:

Remove the admin `ServiceAccount` and `ClusterRoleBinding`

```
#kubectl -n kubernetes-dashboard delete serviceaccount admin-user
#kubectl -n kubernetes-dashboard delete clusterrolebinding admin-user
```