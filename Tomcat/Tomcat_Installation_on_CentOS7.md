# Install OpenJDK1.8:

## Login as root user and install latest OpenJDK1.8:

```shell
#yum -y update
#yum -y install java-1.8* nano wget
```

 

## Confirm Java Version and set the java home:

```shell
#find /usr/lib/jvm/java-1.8* | head -n 3
#cd ~
#nano ~/.bash_profile 
JAVA_HOME=/usr/lib/jvm/java-1.8.0
PATH=$PATH:$HOME/bin:$JAVA_HOME
#source .bash_profile
```



### Check JAVA_HOME:

```shell
#echo $JAVA_HOME
```

 

### Check Java version:

```shell
#java -version
```

# Install Apache Tomcat:

## Check and download Tomcat package at:

Link: https://tomcat.apache.org/download-80.cgi

 

## Download tomcat package:

```shell
#cd /opt
#wget https://mirror.downloadvn.com/apache/tomcat/tomcat-8/v8.5.60/bin/apache-tomcat-8.5.60.tar.gz
#tar -xvzf /opt/apache-tomcat-8.5.60.tar.gz
#mv /opt/apache-tomcat-8.5.60 /opt/tomcat
```

 

## Give executing permissions to startup.sh and shutdown.sh:

```shell
#chmod +x /opt/tomcat/bin/startup.sh /opt/tomcat/bin/shutdown.sh
```

 

## Create link files for tomcat startup.sh and shutdown.sh:

```shell
#ln -s /opt/tomcat/bin/startup.sh /usr/local/bin/tomcatup
#ln -s /opt/tomcat/bin/shutdown.sh /usr/local/bin/tomcatdown
#tomcatup
```

 

## Enable firewall for Tomcat:

```shell
#firewall-cmd --zone=public --permanent --add-port=8080/tcp
#systemctl reload firewalld
```

 

## Check access tomcat application from browser on port 8080

http://192.168.x.x:8080

http://tomcat-192-168-x-x.nip.io:8080

 

## Change Tomcat default port number if run on same server with Jenkins:

```shell
#nano /opt/tomcat/conf/server.xml
```

Update port number in the "`connecter port`"=8090 field in server.xml

Restart tomcat after configuration update

```shell
#tomcatdown
#tomcatup 
```

Enable firewall for Tomcat:

```shell
#firewall-cmd --zone=public --permanent --add-port=8090/tcp
#systemctl reload firewalld
```

 

### Check access tomcat application from browser on port 8080:

http://192.168.x.x:8090

http://tomcat-192-168-x-x.nip.io:8090

 

## Allow login from browser:

### Search for context.xml

```shell
#find / -name context.xml
/opt/tomcat/conf/context.xml
/opt/tomcat/webapps/examples/META-INF/context.xml
/opt/tomcat/webapps/host-manager/META-INF/context.xml
/opt/tomcat/webapps/manager/META-INF/context.xml
```

 

### Comment out value classname fields of all context.xml files in webapps directory. 

```shell
#nano /opt/tomcat/webapps/host-manager/META-INF/context.xml
#nano /opt/tomcat/webapps/manager/META-INF/context.xml
```

Before:

```xml
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
    allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
```

After:

```xml
<!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve"
    allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
```

  

### Update users information in the tomcat-users.xml file:

Edit `tomcat-users.xml` file:

```shell
#nano /opt/tomcat/conf/tomcat-users.xml
```

 Add below users to conf/tomcat-users.xml file before </tomcat-users> tag: 

```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
<user username="admin" password="<password>" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
<user username="deployer" password="<password>" roles="manager-script"/>
<user username="tomcat" password="<password>" roles="manager-gui"/>
```



### Restart tomcat services to effect these changes:

```shell
#tomcatdown
#tomcatup
```

## Check access to Manager App:

Link: http://tomcat-192-168-x-x.nip.io:8080/manager

# Deploy a war file on Tomcat VM using Jenkins

## Install maven Deploy to container plugin without restart

`Manage Jenkins` > `Manage Plugins` > `Available` > search and choose "*Deploy to container*" > `Install without restart`

 

## Create new Jenkins job:

`Jenkins` > `New Item` >

 

Enter an item name

- Item name=Deploy_to_TomcatVM
- Copy from=Build_Maven_project

Description: Build Maven project from Git project and deploy to Tomcat VM

Source Code Management:

Git

- Repository URL=https://github.com/annvt-dl/hello-world.git
- Branch Specifier (blank for 'any')=*/master

Build:

- Root POM=pom.xml
- Goals and options=clean install package

Post-build Actions:

- Choose `Deploy war/ear to a container`

- WAR/EAR files=\*\*/\*.war

- Context path=

- Containers: Tomcat 8.x Remote

- Credentials: `Add` credential:

- - Domain=Global credentials
  - Kind=Username with password
  - Scope=Global
  - Username=deployer
  - Password=<password>
  - ID=deployer
  - Description=User to deploy on Tomcat VM

- Tomcat URL: http://tomcat-192-168-x-x.nip.io:8080

Build Triggers

- Poll SCM > Schedule=* * * * *

`Apply` > `Save` > `Build Now`

 

## Check access to new deployed webapp:

Link: http://tomcat-192-168-x-x.nip.io:8080/webapp
