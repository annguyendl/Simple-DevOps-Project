# Integration Docker with Jenkins:

## Install maven plugin without restart:

On `Jenkins GUI`:

`Manage Jenkins` > `Manage Plugins` > `Available` > search and choose "*Publish over SSH*" > `Install without restart`

 

## Create a user called dockeradmin

On `docker-server`:

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

On `jenkins-server`:

```shell
#ssh dockeradmin@192.168.x.x
#ls
```

 

## Add Publish over SSH info:

On `Jenkins GUI`:

`Manage Jenkins` > `Configure System` > `Publish over SSH` | `SSH Servers` > `Add`

- Name=docker-host
- IP=<192.168.x.x>
- [Advance]
- [X] Use password authentication, or use a different key
- `Test` > `Apply > `Save`

  

# Jenkins Deployment to Docker:

## Create new Jenkins job:

On `Jenkins GUI`:

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

On `docker-server`:

```shell
#ssh dockeradmin@192.168.x.x
#ls
webapp.war
```

 

# Build new Tomcat image included war file:

## Create Dockerfile on Docker server:

On `docker-server`:

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

On `Jenkins GUI`:

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
