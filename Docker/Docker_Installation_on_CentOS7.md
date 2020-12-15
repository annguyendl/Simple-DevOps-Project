# Install Docker CE:

Reference URL : https://docs.docker.com/install/linux/docker-ce/centos/#install-using-the-repository

 

## Install the required packages:

```shell
#yum -y install yum-utils device-mapper-persistent-data lvm2 nano
```

 

## Set up the stable repository:

```shell
#yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

 

## Install the latest version of Docker CE:

```shell
#yum -y install docker-ce
```

Note: If prompted to accept the GPG key, verify that the fingerprint matches 060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35, and if so, accept it.

 

## Enable and start Docker service:

```shell
#systemctl enable docker
#systemctl start docker
#systemctl status docker
```

 

## Verify that docker is installed correctly by running the tomcat image:

```shell
#docker run -d --name tomcat-container -p 8080:8080 tomcat:latest
```

 

## Enable firewall for Tomcat:

```shell
#firewall-cmd --zone=public --permanent --add-port=8080/tcp
#systemctl reload firewalld
```

 

## Fix HTTP 404 on latest image of tomcat:

Due to security reason, the common Manage app isn't deployed in tomcat image so we can't access to link

```shell
#docker exec -it tomcat-container /bin/bash
#cp -R ./webapps.dist/* ./webapps/
```

 

## Check access tomcat container from browser on port 8080:

http://192.168.x.x:8080

http://docker-192-168-x-x.nip.io:8080
