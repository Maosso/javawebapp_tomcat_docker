# Deploy a java app using a Docker container and Tomcat

In this project, I show how to install all dependencies and do a first deploy of a Java app on the Tomcat server out of a Docker container image. I use Maven to manage the build. The app will run on Kubernetes clusters using Minikube. Everything runs on Ubuntu.

## 1. Install platforms and dependencies

### 1.0 Virtual machine and OS
1. dowload a Virtual Machine, for example the [Oracle VM VirtualBox]([https://www.virtualbox.org/](https://www.virtualbox.org/)).
2.  dowload [Ubuntu](https://www.virtualbox.org/). In this tutorial, I use this Ubuntu 64 bit, 18.04 and allocate to my virtual machine 8192 of memory and 20 gb of storage and 2 CPUs. You can decide how much resources to allocate to it in the settings of your virtual machine manager.

### 1.1 Java
```
sudo apt-get install openjdk-8-jdk
```

set up JAVA_HOME in ./bashrc
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin
```
### 1.2 Tomcat
[reference](https://linuxize.com/post/how-to-install-tomcat-9-on-ubuntu-18-04/)

![Tomcat on localhost:8080](https://github.com/Maosso/javawebapp_docker_tomcat/blob/master/tomcat.png)

create a Tomcat user
```
sudo useradd -r -m -U -d /opt/tomcat -s /bin/false tomcat
```
download and untar tomcat; change permissions
```
wget http://www-eu.apache.org/dist/tomcat/tomcat-9/v9.0.36/bin/apache-tomcat-9.0.36.tar.gz -P /tmp
sudo tar xf apache-tomcat-9.0.36.tar.gz
cd /tmp
sudo tar xf apache-tomcat-9.0.36.tar.gz
sudo cp apache-tomcat-9.0.36 /usr/local/
sudo chown -RH tomcat:tomcat /usr/local/apache-tomcat-9.0.36/
sudo sh -c 'chmod +x /usr/local/apache-tomcat-9.0.36/bin/*.sh'
```
start Tomcat
```
cd /usr/local/apache-tomcat-9.0.36
sudo ./bin/startup.sh
```
open
```
sudo vim /usr/local/apache-tomcat-9.0.36/conf/tomcat-users.xml
```
and change roles to
```xml
<tomcat-users>
<!--
    Comments
-->
   <role rolename="manager-gui"/>
<user username="tomcat" password="password" roles="manager-gui"/>
</tomcat-users>
```
![tomcat-user.xml](https://github.com/Maosso/javawebapp_docker_tomcat/blob/master/tomcat-user.xml.png)

finally, use those credential to log into the Tomcat GUI manager at localhost:8080.



### 1.3 Docker Engine
[reference](https://docs.docker.com/engine/install/ubuntu/)
```
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo docker run hello-world
```

### 1.4 Minikube and Kubectl
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s [https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl](https://storage.googleapis.com/kubernetes-release/release/stable.txt%60/bin/linux/amd64/kubectl)
```
Make it binary and executable, move it to path
```
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```
Check that it is installed
```javascript
Kubectl version --client
```

Download minikube, make it binary and executable
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
&& chmod +x minikube
```

Move to my path and install
```
sudo mkdir -p /usr/local/bin/
sudo install minikube
/usr/local/bin/
```
Check that it is installed

### 1.5 MySQL
[reference](https://support.rackspace.com/how-to/install-mysql-server-on-the-ubuntu-operating-system/)
```
apt-get install mysql-server
sudo mysql_secure_installation utility
```
to enter in the mySQL terminal, run
```
sudo mysql -u root -p
```

### 1.6 Git
```
sudo apt install git
```

## Start Minikube Docker pods

### Start Minikube cluster


### Install Docker pods on cluster
Now you need to specify that your Docker image should be installed on the clusters. To do so:
1. create a config.yaml file which specifies the location of the container repository, the MySQL username and password and the port number for docker images;
2. save the .yaml file in the local directory in which you want to later download and deploy the Kubernetes clusters

### Deploy the container in the Kubernetes clusters
1. cd the directory where you want to deploy the container
2. to deploy, use
use Maven to build the app on the Tomcat server:
/home/apps/ShopizerJavaApp$ sudo mvn clean package -e
use Docker to load the app as a docker image in a container:
use Minikube, a local Kubernetes cluster, to manage the container:

![minikube clusters](https://github.com/Maosso/javawebapp_docker_tomcat/blob/master/kubectl%20cluster.png)


 ## Download app and create db
```
git clone https://github.com/nrcandidatelab/ShopizerJavaApp.git
mkdir /home/apps
sudo mv ShopizerJavaApp /home/apps
```
run these commands as specified in database.properties
```
mysql>CREATE DATABASE SALESMANAGER;
mysql>CREATE USER shopizer IDENTIFIED BY 'password';
mysql>GRANT ALL ON SALESMANAGER.* TO shopizer;
mysql>FLUSH PRIVILEGES;
```
![mySQL setup](https://github.com/Maosso/javawebapp_docker_tomcat/blob/master/mysql.png)

in database.properties, append `&usesSSL=false` to `db.jdbcUrl=jdbc:mysql://localhost:3306/SALESMANAGER?autoReconnect=true&useUnicode=true&characterEncoding=UTF-8`

## Compile and deploy

check that your dependencies are installed
![versions](https://github.com/Maosso/javawebapp_docker_tomcat/blob/master/versions.png)

build app on Tomcat server using maven

![mvn outpuut](https://github.com/Maosso/javawebapp_docker_tomcat/blob/master/maven.png)
```
cd /home/apps/ShopizerJavaApp/
sudo mvn clean install
```
load image on the Docker container from the directory that contains the `Dockerfile`
```
cd /home/apps/ShopizerJavaApp/sm-shop
docker build —tag shopizer:1.0 .
```

![docker build](https://github.com/Maosso/javawebapp_docker_tomcat/blob/master/docker%20build.png)

restart Tomcat
```
cd /usr/local/tomcat/bin
sudo ./shutdown
sudo ./startup
```
if you get a 403 access denied error screen, check that ROOT.war is executable.

Your app is now on localhost:8080.

![enter image description hestore](https://github.com/Maosso/javawebapp_docker_tomcat/blob/master/store.png)

Initially, it will have no content; go to `localhost:8080/admin` and add categories to the app:

![add content](https://github.com/Maosso/javawebapp_docker_tomcat/blob/master/add%20content.png)

and create a category tree

![category tree](https://github.com/Maosso/javawebapp_docker_tomcat/blob/master/add%20categories.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTMwOTY4OTE5MSw5MjMwMjcxNTcsLTk1Mj
cxNzgzOSwxNDQwMTMwMzc5LC0yMjAyMjYwOTQsLTg1MjQyMDU5
OSwtMTI4NTgyMjMyLC00NzE0NDQ0OV19
-->