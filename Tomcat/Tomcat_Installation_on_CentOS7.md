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
