# Upgrading to MySQL Enterprise Edition 8.0
## 1. Download binaries
Binaries can be downloaded from the trainer's VM as follow
```
scp opc@10.0.0.203:/home/opc/*.tar.gz /home/opc/
```
Total 4 binaries: mysql-8.0.30-el7-x86_64.tar.gz, mysql-5.7.38-el7-x86_64.tar.gz, mysql-5.6.23-linux-glibc2.5-x86_64.tar.gz, and mariadb-10.9.2-linux-systemd-x86_64.tar.gz </br>
Create new directory
```
mkdir -p /home/opc/archive/8.0 /home/opc/archive/5.7 /home/opc/archive/5.6 /home/opc/archive/mariadb
```
## 2. Upgrading from MySQL Community Edition 8.0
Install MySQL Community 8.0
```
mv /home/opc/mysql-8.0.30-el7-x86_64.tar.gz /home/opc/archive/8.0/
cd /home/opc/archive/8.0
tar -zxvf mysql-8.0.30-el7-x86_64.tar.gz
mkdir -p /home/opc/archive/8.0/db
```
Create opton file (vi /home/opc/archive/8.0/my.cnf)
```
[mysqld]
datadir=/home/opc/archive/8.0/db
binlog-format=ROW
log-bin=/home/opc/archive/8.0/db/bin
port=8000
server_id=10
socket=/home/opc/archive/8.0/db/mysqld.sock
log-error=/home/opc/archive/8.0/db/mysqld.log
```
Create and start database using MySQL 8.0 Community
```
/home/opc/archive/8.0/mysql-8.0.30-el7-x86_64/bin/mysqld --defaults-file=/home/opc/archive/8.0/my.cnf --initialize-insecure
/home/opc/archive/8.0/mysql-8.0.30-el7-x86_64/bin/mysqld_safe --defaults-file=/home/opc/archive/8.0/my.cnf &
```
Load world_x schema
```
/home/opc/archive/8.0/mysql-8.0.30-el7-x86_64/bin/mysql -uroot -h127.0.0.1 -P8000

source /home/opc/software/world_x-db/world_x.sql

show databases;

show plugins;
```
Stop MySQL instance  
```
shutdown;
exit;
```
Start using MySQL Enterprise and check plugin
```
. $HOME/.8030.env

mysqld_safe --defaults-file=/home/opc/archive/8.0/my.cnf &

mysql -uroot -h127.0.0.1 -P8000 -e "show databases"

mysql -uroot -h127.0.0.1 -P8000 -e "show plugins"
```
Shutdown this database to safe server resource
```
mysql -uroot -h127.0.0.1 -P8000 -e "shutdown"
```
## 3. Upgrading from MySQL Community Edition 5.7
### 3.1. Inplace Upgrade
### 3.2. Out of place Upgrade with GTID
### 3.3. Out of place Upgrade with No GTID
## 4. Upgrading from MySQL Community Edition 5.6
### 4.1. Inplace Upgrade
### 4.2. Out of place Upgrade with GTID
### 4.3. Out of place Upgrade with No GTID
## Migrating from MariaDB
