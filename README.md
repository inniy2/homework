# homework
- - - -  
1. Network configuration for first time
```bash
sudo vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```
Change the following:  
> ONBOOT=yes  


```bash
sudo vi /etc/sysconfig/network-scripts/ifcfg-enp0s8
```
Change the following:  
> BOOTPROTO=static  
> ONBOOT=yes   
> IPADDR=192.168.56.40  --> # C clase should be within vbox0.  
> NETMASK=255.255.255.0  
> DNS1=8.8.8.8  
> DNS2=8.8.4.4  


2. Install epel-release
```bash
sudo yum -y update
sudo yum -y wget vim
sudo yum -y install epel-release
```


3. Setup locale
```bash
sudo localectl status
sudo localectl list-locales | grep -i ja
sudo localectl set-locale LANG=ja_JP.utf8
sudo sudo localectl set-locale LANG=ja_JP.utf8
sudo localectl status
```


4. SELinux setup
```bash
sudo sestatus

# set to permissive
sudo setenforce 0

# Change file
sudo vim /etc/selinux/config
```
Change the followings:  
> SELINUX=permissive  



5. Set firewalld  
```bash
sudo firewall-cmd --state

sudo firewall-cmd --get-default-zone

sudo firewall-cmd --zone=public --list-all

sudo firewall-cmd --zone=public --permanent --add-port=80/tcp
sudo firewall-cmd --zone=public --permanent --add-port=8888/tcp
sudo firewall-cmd --zone=public --permanent --add-port=8080/tcp
sudo firewall-cmd --zone=public --permanent --add-port=443/tcp
sudo firewall-cmd --reload

#sudo systemctl restart firewalld
```

6. Install nginx  
```bash
sudo yum -y install nginx  

sudo systemctl enable nginx 

sudo vim /etc/nginx/nginx.conf
```
Change the followings:  
> server {  
>       listen       8888 default_server;  
>       listen       [::]:8888 default_server;  
>  ...


7. Install MariaDB 10.2.16 stable  
```bash
sudo cat << EOF | sudo tee /etc/yum.repos.d/mariadb.repo 
# MariaDB 10.2 CentOS repository list - created 2018-10-17 04:20 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
EOF

sudo yum clean all
sudo yum makecache

sudo yum --showduplicates list MariaDB-server | expand
sudo yum --showduplicates list MariaDB-client | expand


sudo yum install -y MariaDB-server-10.2.16 MariaDB-client-10.2.16

sudo vim /etc/my.cnf.d/server.cnf

```
Change the followings:  
> [mariadb-10.2]  
> character-set-server=utf8  
```bash
sudo systemctl enable maraidb
sudo systemctl start maraidb
```

8. Install OpenJDK 1.8.0  
```bash
sudo yum search openjdk
sudo yum -y install java-1.8.0-openjdk.x86_64 java-1.8.0-openjdk-devel.x86_64
```


9. Install tomcat8  
[Reference 1](https://www.vultr.com/docs/how-to-install-apache-tomcat-8-on-centos-7)  
[Reference 2](https://tomcat.apache.org/download-80.cgi)  
```bash
# Create tomcat user
sudo groupadd tomcat
sudo mkdir /opt/tomcat
sudo useradd -s /bin/nologin -g tomcat -d /opt/tomcat tomcat

# Download and extract
cd /tmp
sudo wget http://ftp.kddilabs.jp/infosystems/apache/tomcat/tomcat-8/v8.5.34/bin/apache-tomcat-8.5.34.tar.gz
sudo tar -zxvf apache-tomcat-8.5.34.tar.gz -C /opt/tomcat --strip-components=1

# Set permission
cd /opt/tomcat
sudo chgrp -R tomcat conf
sudo chmod g+rwx conf
sudo chmod g+r conf/*
sudo chown -R tomcat logs/ temp/ webapps/ work/

sudo chgrp -R tomcat bin
sudo chgrp -R tomcat lib
sudo chmod g+rwx bin
sudo chmod g+r bin/*
sudo cd /opt && sudo chown -R tomcat tomcat/

# Create tomcat.service
sudo vim /etc/systemd/system/tomcat.service
sudo cat << EOF | sudo tee /etc/systemd/system/tomcat.service
[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=forking
SuccessExitStatus=143
PIDFile=/opt/tomcat/temp/tomcat.pid

Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/bin/kill -15 \$MAINPID

User=tomcat
Group=tomcat

[Install]
WantedBy=multi-user.target
EOF

sudo vim  /opt/tomcat/conf/server.xml
```
Change the followings:  
> \<Connector port="8080" protocol="HTTP/1.1"  
>            connectionTimeout="20000"  
>            redirectPort="8443" />  

```bash
sudo systemctl enable tomcat
sudo systemctl start  tomcat
sudo systemctl status tomcat
```


