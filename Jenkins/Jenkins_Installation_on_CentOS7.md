# Install OpenJDK1.8:

## Login as root user and install latest OpenJDK1.8:

```shell
#yum -y update
#yum -y install java-1.8*
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

 

# Install JENKINS:

## Install wget and Jenkins repo:

```shell
#yum -y install wget
#sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
#sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
#yum -y install jenkins
```

 

## Setup Jenkins to start at boot, start jenkins service:

```shell
#chkconfig jenkins on
#systemctl start jenkins
#systemctl status jenkins
```

 

## Enable firewall for Jenkins:

```shell
#firewall-cmd --zone=public --permanent --add-port=8080/tcp
#systemctl reload firewalld
```

##  

## Accessing Jenkins: 

By default Jenkins runs at port 8080, you can access Jenkins at

http://YOUR-SERVER-PUBLIC-IP:8080

http://jenkins-YOUR-SERVER-PUBLIC-IP.nip.io:8080

 

# Configure Jenkins:

The default Username is admin

Grab the default password at

```shell
#cat /var/lib/jenkins/secrets/initialAdminPassword
```

Click [X] on Get started screen to Skip Plugin Installation

 

### Change admin password:

>  [Admin] top menu > Configure > Password

 

### Configure java path:

>  Manage Jenkins > Global Tool Configuration > JDK

- Name=OpenJDK1.8
- JAVA_HOME=/usr/lib/jvm/java-1.8.0

 

## Create another admin user id:

> Manage Jenkins > Manage User > Create User



## Test Jenkins Jobs:

> Jenkins > New Item >

Enter an item name

- Item name=My_First_Project
- Freestyle project

[OK] button

Build 

- Execute shell=echo "Welcome to Jenkins Demo"

[Save] button > [Build Now] icon

Check "Console output"

 

# Configure Git plugin on Jenkins:

## Install Git on Jenkins server:

Install git packages on jenkins server

```shell
#yum install git -y
```

##  

## Setup Git on Jenkins console:

>  Manage Jenkins > Manage Plugins > Available > Search "github" > Select "GitHub" > Click [Install without restart]

Waiting until whole dependency plugins are installed Success

 

## Configure git path:

>  Manage Jenkins > Global Tool Configuration > 

- Name=git
- Path to Git executable=/usr/bin/git

 

# Install Maven on Jenkins server:

## Download maven packages onto Jenkins server:

### Check latest version of Maven:

Link : https://maven.apache.org/download.cgi

 

### Creating maven directory under /opt directory:

```shell
#cd /opt 
#mkdir /opt/maven
#cd /opt/maven
```

 

### Downloading maven version 3.6.*:

```shell
#wget https://downloads.apache.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
#tar -xvzf apache-maven-3.6.3-bin.tar.gz
```

 

## Setup M2_HOME and M2 paths:

Setup M2_HOME and M2 paths in .bash_profile of the user and add these to the path variable

```shell
#nano ~/.bash_profile
M2_HOME=/opt/maven/apache-maven-3.6.3
M2=$M2_HOME/bin
PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2
#source ~/.bash_profile
```

 

Check Maven version:

```shell
#mvn --version
```

 

# Setup Maven on Jenkins:

## Install maven plugin without restart:

> Manage Jenkins > Manage Plugins > Available > "Maven Invoker" > [Install without restart]

> Manage Jenkins > Manage Plugins > Available > "Maven Integration" > [Install without restart]



## Configure maven path:

>  Manage Jenkins > Global Tool Configuration > Maven

Add Maven

- Name=M2_HOME
- MAVEN_HOME=/opt/maven/apache-maven-3.6.3

 

# Test build Maven package:

## New Maven project:

>  Jenkins > New Item > 

Enter an item name

- Item name=Build_Maven_project
- Maven project

> [OK]

Description: Build Maven project from Git project

Source Code Management:

Git

- Repository URL=https://github.com/annvt-dl/hello-world.git
- Branch Specifier (blank for 'any')=*/master

Build:

- Root POM=pom.xml
- Goals and options=clean install package

> [Apply] > [Save] > Build Now

  

## Check build result:

```shell
#cd /var/lib/jenkins/workspace/Build_Maven_project
#ls -l
```

