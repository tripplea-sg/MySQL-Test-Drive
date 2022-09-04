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
gtid_mode=on
enforce-gtid-consistency
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
Shutdown this database and delete its files
```
mysql -uroot -h127.0.0.1 -P8000 -e "shutdown"

rm -Rf /home/opc/archive/8.0/db/*
```
## 3. Upgrading from MySQL Community Edition 5.7
### 3.1. Inplace Upgrade
Install MySQL Community 5.7
```
mv /home/opc/mysql-5.7.38-el7-x86_64.tar.gz /home/opc/archive/5.7/
cd /home/opc/archive/5.7
tar -zxvf mysql-5.7.38-el7-x86_64.tar.gz
mkdir -p /home/opc/archive/5.7/db
```
Create opton file (vi /home/opc/archive/5.7/my.cnf)
```
[mysqld]
datadir=/home/opc/archive/5.7/db
binlog-format=ROW
log-bin=/home/opc/archive/5.7/db/bin
port=5700
server_id=20
socket=/home/opc/archive/5.7/db/mysqld.sock
log-error=/home/opc/archive/5.7/db/mysqld.log
gtid_mode=on
enforce-gtid-consistency
```
Create and start database using MySQL 8.0 Community
```
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysqld --defaults-file=/home/opc/archive/5.7/my.cnf --initialize-insecure
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysqld_safe --defaults-file=/home/opc/archive/5.7/my.cnf &
```
Load world_x schema
```
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysql -uroot -h127.0.0.1 -P5700

source /home/opc/software/world_x-db/world_x.sql

show databases;

show plugins;

exit;
```
Check Compatibility with MySQL 8.0
```
mysqlsh root@localhost:5700 -- util checkForServerUpgrade
```
If everything looks okay, then continue with upgrade. </br>
Stop MySQL server
```
mysql -uroot -h127.0.0.1 -P5700 -e "shutdown"
```
Upgrade to MySQL EE 8.0
```
. $HOME/.8030.env

mysqld_safe --defaults-file=/home/opc/archive/5.7/my.cnf &

mysql -uroot -h127.0.0.1 -P5700 -e "show databases"

mysql -uroot -h127.0.0.1 -P5700 -e "show plugins"
```
Shutdown this database and delete files
```
mysql -uroot -h127.0.0.1 -P5700 -e "shutdown"

rm -Rf /home/opc/archive/5.7/db/*
```
### 3.2. Out of place Upgrade with GTID
Create and start database using MySQL 8.0 Community
```
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysqld --defaults-file=/home/opc/archive/5.7/my.cnf --initialize-insecure
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysqld_safe --defaults-file=/home/opc/archive/5.7/my.cnf &
```
Load world_x schema
```
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysql -uroot -h127.0.0.1 -P5700

source /home/opc/software/world_x-db/world_x.sql

show databases;

show plugins;

show variables like '%gtid%';

exit;
```
Backup database 5.7 using MySQL Shell
```
mkdir -p /home/opc/archive/5.7/backup

mysqlsh root@localhost:5700 -- util dumpInstance /home/opc/archive/5.7/backup

ls /home/opc/archive/5.7/backup

cat /home/opc/archive/5.7/backup/@.json
```
Create database 8.0 using MySQL EE
```
. $HOME/.8030.env

mysqld --defaults-file=/home/opc/archive/8.0/my.cnf --initialize-insecure

mysqld_safe --defaults-file=/home/opc/archive/8.0/my.cnf &
```
Restore from backup to MySQL EE 8.0
```
mysql -uroot -h127.0.0.1 -P8000 -e "set global local_infile=on"

mysqlsh root@localhost:8000 -- util loadDump /home/opc/archive/5.7/backup --ignoreVersion
```
Create replication user on mysql 5.7
```
mysql -uroot -h127.0.0.1 -P5700 -e "create user repl@'%' identified by 'repl'; grant replication slave on *.* to repl@'%';"
```
Create MySQL Replication from 5.7 to MySQL 8.0
```
mysql -uroot -h127.0.0.1 -P8000 -e "change master to master_user='repl', master_host='127.0.0.1', master_port=5700, master_password='repl', master_auto_position=1 for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P8000 -e "start slave for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P8000 -e "show slave status for channel 'channel1' \G"
```

### 3.3. Out of place Upgrade with No GTID
## 4. Upgrading from MySQL Community Edition 5.6
### 4.1. Inplace Upgrade
### 4.2. Out of place Upgrade with GTID
### 4.3. Out of place Upgrade with No GTID
## Migrating from MariaDB
