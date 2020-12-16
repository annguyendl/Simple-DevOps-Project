# Install kubectl:

## Get latest version of kubectl:

```shell
#KUBE_VERSION=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)
#echo $KUBE_VERSION
#cd /opt/
#curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/${KUBE_VERSION}/bin/linux/amd64/kubectl
#chmod +x kubectl
#mv -f kubectl /usr/local/bin/
#kubectl version
```

 It will show error here because no kubenetes' cluster has configured.

## Install kubectl auto-completion:

### Install bash-completion:

```sh
#yum -y install bash-completion
```

Above command creates the file in `/usr/share/bash-completion/bash_completion`.

Check bash-completion:

```sh
#type _init_completion
```

If above command didn't show any result, add `source` command to `.bashrc` file and check again.

```sh
#nano ~/.bashrc
source /usr/share/bash-completion/bash_completion
#source ~/.bashrc
```

### Install kubectl auto completion scripts:

Add permanent auto-complete scripts when login:

```sh
#echo 'source <(kubectl completion bash)' >> ~/.bashrc
#source ~/.bashrc
```

OR we can add kubectl scripts to bash-completion scripts folder:

```sh
#kubectl completion bash > /etc/bash_completion.d/kubectl 
```

Add an alias for kubectl (optional):

```sh
#nano ~/.bashrc
alias k=kubectl
complete -F __start_kubectl k
#source ~/.bashrc
```

### kubectl's Cheat sheet:

Refer Link: https://kubernetes.io/docs/reference/kubectl/cheatsheet/