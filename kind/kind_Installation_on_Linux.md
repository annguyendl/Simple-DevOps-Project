# Install `kind` on Linux

## Pre-requisite:

- docker ce

## Install `go` on Linux (CentOS7/ Ubuntu18):

Refer to https://golang.org/doc/install

Download latest version of go:

```sh
#wget https://golang.org/dl/go1.15.6.linux-amd64.tar.gz
```

Extract to /usr/local

```sh
#sudo tar -C /usr/local -xvzf go1.15.6.linux-amd64.tar.gz
```

Add /usr/local/go/bin to the `PATH` environment variable.

### CentOS7

```sh
#nano ~/.bash_profile
PATH=$PATH:$HOME/bin:/usr/local/go/bin
#source ~/.bash_profile
```

### Ubuntu:

```shell
#nano ~/.profile
PATH=$PATH:$HOME/bin:/usr/local/go/bin
#source ~/.profile
```

Verify that you've installed Go by opening a command prompt and typing the following command:

```shell
#go version
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

## Install kind:

Download kind:

```bash
#sudo curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.9.0/kind-linux-amd64
#sudo chmod +x ./kind
#sudo mv ./kind /usr/local/bin/kind
```

Create a kind cluster

```shell
#sudo kind create cluster
```

Some other kind commands:

```shell
## Default cluster context name is `kind`.
#sudo kind create cluster --name kind-2

#sudo kind get clusters
kind
kind-2

#sudo kind delete cluster
```



Interact with a specific cluster, you only need to specify the cluster name as a context in kubectl:

```shell
#kubectl cluster-info --context kind-kind
```

