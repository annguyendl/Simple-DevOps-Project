# Ansible Installation

Ansible is an open-source automation platform. It is very, very simple to set up and yet powerful. Ansible can help you with configuration management, application deployment, task automation.

### Pre-requisites

1. An AWS EC2 instance (on Control node)
2. Installed `docker`

### Installation steps:
#### on Amazon EC2 instance of `ansible-server`

Install python and python-pip
```sh
#yum -y install python
#yum -y install python-pip
```

Install ansible using pip check for version
```sh
#pip install ansible
#ansible --version
```

Create a user called `ansadmin` on both Control node (`ansible-server`) and Managed hosts  

```sh
#useradd ansadmin
#passwd ansadmin
```

Below command grant `sudo` access to `ansadmin` user. But we strongly recommended using "visudo" command if you are aware vi or nano editor.  (on Control node and Managed host)
```sh
#echo "ansadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
```

Log in as a ansadmin user on Control node (`ansible-server`) and generate ssh key 
```sh 
#sudo su - ansadmin
#ssh-keygen
```

Copy keys onto all ansible managed hosts (on Control node)
```sh 
#ssh-copy-id ansadmin@<target-server>
```

Ansible server used to create images and store on docker registry. Hence install docker, start docker services and add ansadmin to the docker group. 
```sh
#yum -y install docker

## start docker services at host start 
#systemctl enable docker

## start docker services 
#systemctl start docker

##check docker services status 
#systemctl status docker

```

Add ansadmin to docker group

```sh
#usermod -aG docker ansadmin
#id ansadmin
```

Create a directory `/etc/ansible` and create an inventory file called "`/etc/ansible/hosts`" add control node and managed hosts IP addresses to it.

   ```sh
#mkdir /etc/ansible
#nano /etc/ansible/hosts
localhost ansible_connection=local
managed-hosts-IP ansible_user=ansadmin   
   ```

Validation test

```sh
#ansible all -m ping
```