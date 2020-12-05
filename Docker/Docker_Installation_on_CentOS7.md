# Install Docker CE:

Reference URL : https://docs.docker.com/install/linux/docker-ce/centos/#install-using-the-repository

 

## Install the required packages:

```shell
#yum -y install yum-utils device-mapper-persistent-data lvm2 nano
```

 

## Set up the stable repository:

```shell
#yum-config-manager --add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
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

 

# Integration Docker with Jenkins:

## Install maven plugin without restart:

`Manage Jenkins` > `Manage Plugins` > `Available` > search and choose "*Publish over SSH*" > `Install without restart`

 

## Create a user called dockeradmin

```shell
#useradd dockeradmin
#passwd dockeradmin
```

#### Add a user to docker group to manage docker

```shell
#cat /etc/group
#usermod -aG docker dockeradmin
```

#### Check dockeradmin was added to docker group:

```shell
#id dockeradmin
```

### Enable login via user/password:

```shell
#nano /etc/ssh/sshd_config
PasswordAuthentication yes

#systemctl restart sshd
```

 

## Check ssh from Jenkins server:

```shell
#ssh dockeradmin@192.168.x.x
#ls
```

 

## Add Publish over SSH info:

`Manage Jenkins` > `Configure System` > `Publish over SSH` | `SSH Servers` > `Add`

- Name=docker-host
- IP=<192.168.x.x>
- [Advance]
- [X] Use password authentication, or use a different key
- `Test` > `Apply > `Save`

  

# Jenkins Deployment to Docker:

## Create new Jenkins job:

`Jenkins` > `New Item` >

 

Enter an item name

- Item name=Deploy_to_Docker
- Copy from=Build_Maven_project

Description: Build Maven project from Git project and deploy to Docker

Source Code Management:

Git

- Repository URL=https://github.com/annvt-dl/hello-world.git
- Branch Specifier (blank for 'any')=*/master

Build:

- Root POM=pom.xml
- Goals and options=clean install package

Post-build Actions: `Send Build artifacts via SSH/SSH Publisher`

- SSH ServerName=docker-host
- Transfer set/Source files=webapp/target/*.war
- Remove prefix=webapp/target
- Remote directory=`.`

`Apply` > `Save` > `Build Now`

 

## Check war file was deployed to Docker server:

```shell
#ssh dockeradmin@192.168.x.x
#ls
webapp.war
```

 

# Build new Tomcat image included war file:

## Create Dockerfile on Docker server:

### Login to dockeradmin:

```shell
#sudo su - dockeradmin
#nano Dockerfile
FROM tomcat:latest
MAINTAINER annvt@unit.com.vn
COPY ./webapp.war /usr/local/tomcat/webapps
```

 

### Remove running container:

```shell
#docker ps
#docker stop tomcat-container
#docker rm tomcat-container
```

 

### Build new Tomcat image:

```shell
#docker build -t devops-project .
#docker images
```

 

### Test new Tomcat image:

```shell
#docker run -d --name tomcat-container -p 8080:8080 devops-project
```

 

### Check access to container:

http://192.168.x.x:8080/webapp

http://docker-192-168-x-x.nip.io:8080/webapp

  

## Update Jenkins Deployment to Docker:

`Jenkins` > [Deploy_to_Docker] > `Configure`

Post-build Actions: Send Build artifacts via SSH/SSH Publisher

- Exec command=

> cd /home/dockeradmin/;
>
> docker stop tomcat-container;
>
> docker rm tomcat-container;
>
> docker build -t devops-image .;
>
> docker run -d --name tomcat-container -p 8080:8080 devops-image;

`Apply` > `Save` > `Build Now`



### Check access to container:

http://192.168.x.x:8080/webapp

http://docker-192-168-x-x.nip.io:8080/webapp
