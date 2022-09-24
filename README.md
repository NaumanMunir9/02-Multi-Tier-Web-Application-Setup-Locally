# Prerequisites

- JDK 1.8 or later
- Maven 3 or later
- MySQL 5.6 or later

## Technologies

- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- MySQL

## Database

Here,we used Mysql DB

MSQL DB Installation Steps for Linux ubuntu 14.04:

- $ sudo apt-get update
- $ sudo apt-get install mysql-server

Then look for the file :

- /src/main/resources/accountsdb
- accountsdb.sql file is a mysql dump file.we have to import this dump to mysql db server
- > mysql -u <user_name> -p accounts < accountsdb.sql

---

## Setup DB

```shell

vagrant up

vagrant ssh db01

sudo -i

yum update -y

vi /etc/profile > DATABASE_PASS='admin123'

source /etc/profile

yum install epel-release -y

yum install git mariadb-server -y

systemctl start mariadb

systemctl enable mariadb

systemctl status mariadb

mysql_secure_installation

- set root password Y
- remove anonymous users Y
- disallow root login remotely n
- remove test database and access to it Y
- reload privilege table now Y

mysql -u root -p 

- exit

git clone [URL]

cd [folder]/src/main/resources

mysql -u root -p"$DATABASE_PASS" -e "create database accounts"

cd ../../..

mysql -u root -p"$DATABASE_PASS" accounts < src/main/resources/db_backup.sql

mysql -u root -p"$DATABASE_PASS" -e "FLUSH PRIVILEGES"

mysql -u root -p"$DATABASE_PASS"

- show databases;
- use accounts;
- show tables;
- exit

```

## Setup Memcached

```shell

sudo -i

yum update -y

yum install epel-release -y

yum install memcached -y

systemctl start memcached

systemctl enable memcached

systemctl status memcached

memcached -p 11211 -U 11111 -u memcached -d

ss -tunlp | grep 11211

```

## Setup Rabbit MQ

```shell

sudo -i

yum update -y

yum install epel-release -y

sudo yum install wget -y

cd /tmp/

wget http://packages.erlang-solutions.com/erlang-solutions-2.0-1.noarch.rpm

sudo rpm -Uvh erlang-solutions-2.0-1.noarch.rpm

sudo yum -y install erlang socat

curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash

sudo yum install rabbitmq-server -y

sudo systemctl start rabbitmq-server

sudo systemctl enable rabbitmq-server

sudo systemctl status rabbitmq-server

# Config Change
sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'

sudo rabbitmqctl add_user test test

sudo rabbitmqctl set_user_tags test administratorRestart RabbitMQ service

systemctl restart rabbitmq-server

```

## Setup Tomcat Server

```shell

vagrant ssh app01

sudo -i

yum update -y

yum install epel-release -y

# Install Dependencies
yum install java-1.8.0-openjdk -y

yum install git maven wget -y

# Change dir to /tmp
cd /tmp/

# Download & Extract Tomcat Package
wget https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.37/bin/apache-tomcat-8.5.37.tar.gz

tar xzvf apache-tomcat-8.5.37.tar.gz

# Add tomcat user
useradd --home-dir /usr/local/tomcat8 --shell /sbin/nologin tomcat

# Copy data to tomcat home dir
cp -r /tmp/apache-tomcat-8.5.37/* /usr/local/tomcat8/

# Make tomcat user owner of tomcat home dir
chown -R tomcat.tomcat /usr/local/tomcat8

# Setup systemd for tomcat
# Update file with following content.
vi /etc/systemd/system/tomcat.service

[Unit]
Description=Tomcat
After=network.target

[Service]
User=tomcat
WorkingDirectory=/usr/local/tomcat8
Environment=JRE_HOME=/usr/lib/jvm/jre
Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_HOME=/usr/local/tomcat8
Environment=CATALINE_BASE=/usr/local/tomcat8
ExecStart=/usr/local/tomcat8/bin/catalina.sh run
ExecStop=/usr/local/tomcat8/bin/shutdown.sh
SyslogIdentifier=tomcat-%i

[Install]
WantedBy=multi-user.target

systemctl daemon-reload
systemctl start tomcat
systemctl enable tomcat
systemctl status tomcat

# CODE BUILD & DEPLOY (app01)

# Download Source code
git clone -b local-setup https://github.com/devopshydclub/vprofile-project.git

# Update configuration
cd vprofile-project
vim src/main/resources/application.properties
# Update file with backend server details

# Build code
# Run below command inside the repository (vprofile-project)
mvn install

# Deploy artifact
systemctl stop tomcat
sleep 120
rm -rf /usr/local/tomcat8/webapps/ROOT*
cp target/vprofile-v2.war /usr/local/tomcat8/webapps/ROOT.war
ls /usr/local/tomcat8/webapps/
systemctl start tomcat
sleep 300

chown tomcat.tomcat usr/local/tomcat8/webapps -R
systemctl restart tomcat

# NGINX SETUP
# Login to the Nginx vm

vagrant ssh web01

sudo -i

# Verify Hosts entry, if entries missing update the it with IP and hostnames

cat /etc/hosts

#Update OS with latest patches

apt update && apt upgrade -y

Install nginx

apt install nginx -y

# Create Nginx conf file with below content
vi /etc/nginx/sites-available/vproapp

upstream vproapp {
server app01:8080;
}
server {
	listen 80;
location / {
	proxy_pass http://vproapp;
}
}

#Remove default nginx conf

rm -rf /etc/nginx/sites-enabled/default

# Create link to activate website

ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp

# Restart Nginx

systemctl restart nginx

```
