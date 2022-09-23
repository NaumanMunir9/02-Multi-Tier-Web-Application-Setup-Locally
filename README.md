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
