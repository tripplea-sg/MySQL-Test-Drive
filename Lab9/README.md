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
Create and start database using MySQL 5.7 Community
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
Stop database 8.0 and remove instance
```
mysql -uroot -h127.0.0.1 -P8000 -e "shutdown"

rm -Rf /home/opc/archive/8.0/db/*
```
Stop and remove database instance 5.7
```
mysql -uroot -h127.0.0.1 -P5700 -e "shutdown"
rm -Rf /home/opc/archive/5.7/db/*
```
### 3.3. Out of place Upgrade with No GTID
Create option file for starting database in No GTID mode
```
cat /home/opc/archive/5.7/my.cnf | grep -v gtid > /home/opc/archive/5.7/my-nogtid.cnf
```
Start database 5.7 with no GTID
```
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysqld --defaults-file=/home/opc/archive/5.7/my-nogtid.cnf --initialize-insecure
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysqld_safe --defaults-file=/home/opc/archive/5.7/my-nogtid.cnf &
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysql -uroot -h127.0.0.1 -P5700 -e "source /home/opc/software/world_x-db/world_x.sql"
```
Check binlog position
```
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysql -uroot -h127.0.0.1 -P5700 -e "show master status"
```
Backup database
```
rm -Rf /home/opc/archive/5.7/backup/*
mysqlsh root@localhost:5700 -- util dumpInstance /home/opc/archive/5.7/backup
```
Create MySQL 8.0 EE
```
. $HOME/.8030.env
mysqld --defaults-file=/home/opc/archive/8.0/my.cnf --initialize-insecure
mysqld_safe --defaults-file=/home/opc/archive/8.0/my.cnf &
```
Restore database
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
mysql -uroot -h127.0.0.1 -P8000 -e "change master to master_host='127.0.0.1', master_port=5700, master_user='repl', master_password='repl', master_log_file='bin.000002', master_log_pos=911056, ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS=LOCAL for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P8000 -e "start slave for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P8000 -e "show slave status for channel 'channel1' \G"
```
Making transaction on MySQL 5.7
```
mysql -uroot -h127.0.0.1 -P5700 -e "create database test;"
mysql -uroot -h127.0.0.1 -P5700 -e "create user apps@'%' identified by 'apps';"
```
Check transaction on MySQL 8.0
```
mysql -uroot -h127.0.0.1 -P8000 -e "show databases;"
mysql -uroot -h127.0.0.1 -P8000 -e "select user, host from mysql.user"
mysql -uroot -h127.0.0.1 -P5600 -e "select @@version"
```
Stop database 8.0 and remove instance
```
mysql -uroot -h127.0.0.1 -P8000 -e "shutdown"

rm -Rf /home/opc/archive/8.0/db/*
```
Stop and remove database instance 5.7
```
mysql -uroot -h127.0.0.1 -P5700 -e "shutdown"
rm -Rf /home/opc/archive/5.7/db/*
```
## 4. Upgrading from MySQL Community Edition 5.6
### 4.1. Inplace Upgrade
Install MySQL Community 5.6
```
mv /home/opc/mysql-5.6.23-linux-glibc2.5-x86_64.tar.gz /home/opc/archive/5.6/
cd /home/opc/archive/5.6
tar -zxvf mysql-5.6.23-linux-glibc2.5-x86_64.tar.gz
mkdir -p /home/opc/archive/5.6/db
sudo yum install perl perl-Data-Dumper
sudo mkdir -p /usr/local/mysql/share
sudo chown -Rf opc:opc /usr/local/mysql
ln -s /home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/share/english/errmsg.sys /usr/local/mysql/share/errmsg.sys
```
Create opton file (vi /home/opc/archive/5.6/my.cnf)
```
[mysqld]
datadir=/home/opc/archive/5.6/db
binlog-format=ROW
log-bin=/home/opc/archive/5.6/db/bin
port=5600
server_id=30
socket=/home/opc/archive/5.6/db/mysqld.sock
log-error=/home/opc/archive/5.6/db/mysqld.log
gtid_mode=on
enforce-gtid-consistency
log_slave_updates = TRUE
```
Create and start database
```
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/scripts/mysql_install_db --defaults-file=/home/opc/archive/5.6/my.cnf --basedir=/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64

/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysqld --defaults-file=/home/opc/archive/5.6/my.cnf &
```
Load sakila schema
```
wget https://downloads.mysql.com/docs/sakila-db.zip
unzip sakila-db.zip 
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "source /home/opc/archive/5.6/sakila-db/sakila-schema.sql"
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "source /home/opc/archive/5.6/sakila-db/sakila-data.sql"
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "show databases"
```
Upgrade database to MySQL 5.7 Community
```
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysqladmin -uroot -h127.0.0.1 -P5600 shutdown

/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysqld --defaults-file=/home/opc/archive/5.6/my.cnf &

/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysql_upgrade -uroot -h127.0.0.1 -P5600

/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "shutdown"
```
Database is upgraded to MySQL 5.7. Now, Upgrade to MySQL EE 8.0
```
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "shutdown"

. $HOME/.8030.env

mysqld_safe --defaults-file=/home/opc/archive/5.6/my.cnf &

mysql -uroot -h127.0.0.1 -P5600 -e "show plugins"
mysql -uroot -h127.0.0.1 -P5600 -e "select @@version"
```
Stop and delete database
```
mysql -uroot -h127.0.0.1 -P5600 -e "shutdown"
```
### 4.2. Out of place Upgrade with GTID
Create and start database
```
rm -Rf /home/opc/archive/5.6/db/*
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/scripts/mysql_install_db --defaults-file=/home/opc/archive/5.6/my.cnf --basedir=/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64

/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysqld --defaults-file=/home/opc/archive/5.6/my.cnf &
```
Load sakila schema
```
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "source /home/opc/archive/5.6/sakila-db/sakila-schema.sql"
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "source /home/opc/archive/5.6/sakila-db/sakila-data.sql"
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "show databases"
```
Backup MySQL 5.6
```
mkdir -p /home/opc/archive/5.6/backup
mysqlsh root@localhost:5600 -- util dumpInstance '/home/opc/archive/5.6/backup'
```
Create and start database using MySQL 5.7 Community
```
rm -Rf /home/opc/archive/5.7/db/*
echo "master_info_repository=TABLE" >> /home/opc/archive/5.7/my.cnf
echo "relay_log_info_repository=TABLE" >> /home/opc/archive/5.7/my.cnf
echo "log_slave_updates=TRUE" >> /home/opc/archive/5.7/my.cnf
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysqld --defaults-file=/home/opc/archive/5.7/my.cnf --initialize-insecure
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysqld_safe --defaults-file=/home/opc/archive/5.7/my.cnf &
```
Restore database backup to MySQL 5.7 Community 
```
mysqlsh root@localhost:5700 -- util loadDump /home/opc/archive/5.6/backup --ignoreVersion
```
Create replication user on 5.6
```
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "create user repl@'localhost'; set password for repl@'localhost' = password('repl'); grant replication slave on *.* to repl@'localhost';"
```
Create replication channel on MySQL 5.7 pointing to MySQL 5.6
```
mysql -uroot -h127.0.0.1 -P5700 -e "change master to master_user='repl', master_host='127.0.0.1', master_port=5600, master_password='repl', master_auto_position=1 for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P5700 -e "start slave for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P5700 -e "show slave status for channel 'channel1' \G"
```
Backup database 5.7
```
mysql -uroot -h127.0.0.1 -P5700 -e "stop slave for channel 'channel1';"
rm -Rf /home/opc/archive/5.7/backup/*
mysqlsh root@localhost:5700 -- util dumpInstance /home/opc/archive/5.7/backup
```
Create and start database using MySQL 8.0 Enterprise
```
rm -Rf /home/opc/archive/8.0/db/*
. $HOME/.8030.env
mysqld --defaults-file=/home/opc/archive/8.0/my.cnf --initialize-insecure
mysqld_safe --defaults-file=/home/opc/archive/8.0/my.cnf &
```
Restore from backup of MySQL 5.7 to MySQL 8.0 Enterprise
```
mysql -uroot -h127.0.0.1 -P8000 -e "set global local_infile=on"
mysqlsh root@localhost:8000 -- util loadDump /home/opc/archive/5.7/backup --ignoreVersion
```
Create replication channel on MySQL 8.0 Enterprise pointing to MySQL 5.7 Community
```
mysql -uroot -h127.0.0.1 -P8000 -e "change master to master_user='repl', master_host='127.0.0.1', master_port=5700, master_password='repl', master_auto_position=1 for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P8000 -e "start slave for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P8000 -e "show slave status for channel 'channel1' \G"

mysql -uroot -h127.0.0.1 -P8000 -e "stop slave for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P8000 -e "set global gtid_purged='a0c01bc0-2d2a-11ed-93d1-02001700441b:66-94'"

mysql -uroot -h127.0.0.1 -P8000 -e "start slave for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P8000 -e "show slave status for channel 'channel1' \G"
```
Start replication on MySQL 5.7
```
mysql -uroot -h127.0.0.1 -P5700 -e "start slave for channel 'channel1';"
mysql -uroot -h127.0.0.1 -P5700 -e "show slave status for channel 'channel1' \G"
```
Create transaction on MySQL 5.6
```
mysql -uroot -h127.0.0.1 -P5600 -e "create database dev; create table dev.test (i int); insert into dev.test values (1), (2), (3);"
```
Check replication result
```
mysql -uroot -h127.0.0.1 -P5600 -e "select @@version"

mysql -uroot -h127.0.0.1 -P5700 -e "select @@version; select * from dev.test;"

mysql -uroot -h127.0.0.1 -P8000 -e "select @@version; select * from dev.test;"
```
Shutdown and remove all instances
```
mysql -uroot -h127.0.0.1 -P8000 -e "shutdown"
mysql -uroot -h127.0.0.1 -P5700 -e "shutdown"
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysqladmin -uroot -h127.0.0.1 -P5600 shutdown

rm -Rf /home/opc/archive/5.6/db/* /home/opc/archive/5.7/db/* /home/opc/archive/8.0/db/*
```
### 4.3. Out of place Upgrade with No GTID
Create option file without gtid
```
cat /home/opc/archive/5.6/my.cnf | grep -v gtid > /home/opc/archive/5.6/my-nogtid.cnf
```
Create and start database
```
rm -Rf /home/opc/archive/5.6/db/*
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/scripts/mysql_install_db --defaults-file=/home/opc/archive/5.6/my-nogtid.cnf --basedir=/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64

/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysqld --defaults-file=/home/opc/archive/5.6/my-nogtid.cnf &
```
Load sakila schema
```
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "source /home/opc/archive/5.6/sakila-db/sakila-schema.sql"
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "source /home/opc/archive/5.6/sakila-db/sakila-data.sql"
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "show databases"
```
Backup MySQL 5.6
```
rm /home/opc/archive/5.6/backup/*
mysqlsh root@localhost:5600 -- util dumpInstance '/home/opc/archive/5.6/backup'
```
Check binlog position
```
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "show master status"
```
Create and start database using MySQL 5.7 Community
```
rm -Rf /home/opc/archive/5.7/db/*
echo "master_info_repository=TABLE" >> /home/opc/archive/5.7/my-nogtid.cnf
echo "relay_log_info_repository=TABLE" >> /home/opc/archive/5.7/my-nogtid.cnf
echo "log_slave_updates=TRUE" >> /home/opc/archive/5.7/my-nogtid.cnf
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysqld --defaults-file=/home/opc/archive/5.7/my-nogtid.cnf --initialize-insecure
/home/opc/archive/5.7/mysql-5.7.38-el7-x86_64/bin/mysqld_safe --defaults-file=/home/opc/archive/5.7/my-nogtid.cnf &
```
Restore database backup to MySQL 5.7 Community 
```
mysqlsh root@localhost:5700 -- util loadDump /home/opc/archive/5.6/backup --ignoreVersion
```
Create replication user on 5.6
```
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysql -uroot -h127.0.0.1 -P5600 -e "create user repl@'localhost'; set password for repl@'localhost' = password('repl'); grant replication slave on *.* to repl@'localhost';"
```
Create MySQL Replication from 5.6 to MySQL 5.7
```
mysql -uroot -h127.0.0.1 -P5700 -e "change master to master_host='127.0.0.1', master_port=5600, master_user='repl', master_password='repl', master_log_file='bin.000003', master_log_pos=1337390 for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P5700 -e "start slave for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P5700 -e "show slave status for channel 'channel1' \G"
```
Stop Replication and backup MySQL 5.7
```
mysql -uroot -h127.0.0.1 -P5700 -e "stop slave for channel 'channel1';"

rm -Rf /home/opc/archive/5.7/backup/*
mysqlsh root@localhost:5700 -- util dumpInstance /home/opc/archive/5.7/backup
```
Create database with MySQL 8.0 Enterprise and restore from backup
```
rm -Rf /home/opc/archive/8.0/db/*
. $HOME/.8030.env
mysqld --defaults-file=/home/opc/archive/8.0/my.cnf --initialize-insecure
mysqld_safe --defaults-file=/home/opc/archive/8.0/my.cnf &
mysql -uroot -h127.0.0.1 -P8000 -e "set global local_infile=on"
mysqlsh root@localhost:8000 -- util loadDump /home/opc/archive/5.7/backup --ignoreVersion
```
Get binlog last position from MySQL 5.7
```
mysql -uroot -h127.0.0.1 -P5700 -e "show master status"
```
Create replication channel on MySQL 8.0 Enterprise
```
mysql -uroot -h127.0.0.1 -P8000 -e "change master to master_host='127.0.0.1', master_port=5700, master_user='repl', master_password='repl', master_log_file='bin.000003', master_log_pos=154, ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS=LOCAL for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P8000 -e "start slave for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P8000 -e "show slave status for channel 'channel1' \G"
```
Create transaction on MySQL 5.6
```
mysql -uroot -h127.0.0.1 -P5600 -e "create database dev; create table dev.test (i int); insert into dev.test values (1), (2), (3);"
```
Check replication result
```
mysql -uroot -h127.0.0.1 -P5700 -e "start slave for channel 'channel1';"

mysql -uroot -h127.0.0.1 -P5600 -e "select @@version"

mysql -uroot -h127.0.0.1 -P5700 -e "select @@version; select * from dev.test;"

mysql -uroot -h127.0.0.1 -P8000 -e "select @@version; select * from dev.test;"
```
Shutdown and remove all instances
```
mysql -uroot -h127.0.0.1 -P8000 -e "shutdown"
mysql -uroot -h127.0.0.1 -P5700 -e "shutdown"
/home/opc/archive/5.6/mysql-5.6.23-linux-glibc2.5-x86_64/bin/mysqladmin -uroot -h127.0.0.1 -P5600 shutdown

rm -Rf /home/opc/archive/5.6/db/* /home/opc/archive/5.7/db/* /home/opc/archive/8.0/db/*
```
## 5. Migrating from MariaDB
Install MariaDB
```
cd /home/opc
mv mariadb-10.9.2-linux-systemd-x86_64.tar.gz archive/mariadb/

cd archive/mariadb/
tar -zxvf mariadb-10.9.2-linux-systemd-x86_64.tar.gz
mkdir db
```
Create option file for mariadb (vi /home/opc/archive/mariadb/my.cnf)
```
[mysqld]
datadir=/home/opc/archive/mariadb/db
binlog-format=ROW
log-bin=/home/opc/archive/mariadb/db/bin
port=3306
server_id=50
socket=/home/opc/archive/mariadb/db/mysqld.sock
log-error=/home/opc/archive/mariadb/db/mysqld.log
```
Create mariadb database
```
/home/opc/archive/mariadb/mariadb-10.9.2-linux-systemd-x86_64/scripts/mysql_install_db --defaults-file=my.cnf
/home/opc/archive/mariadb/mariadb-10.9.2-linux-systemd-x86_64/bin/mysqld_safe --defaults-file=my.cnf --skip-grant-table &

mysql -uroot -h127.0.0.1 -e "flush privileges; set password for root@'localhost'=password('root'); shutdown;"
/home/opc/archive/mariadb/mariadb-10.9.2-linux-systemd-x86_64/bin/mysqld_safe --defaults-file=my.cnf &

mysql -uroot -h127.0.0.1 -P3306 -proot -e "source /home/opc/archive/5.6/sakila-db/sakila-schema.sql"
mysql -uroot -h127.0.0.1 -P3306 -proot -e "source /home/opc/archive/5.6/sakila-db/sakila-data.sql"
```
Migrate to MySQL 8.0 Enterprise Edition
```
mysqlsh root@localhost:3306 -- util dumpInstance /home/opc/archive/mariadb/backup

. $HOME/.8030.env
mysqld --defaults-file=/home/opc/archive/8.0/my.cnf --initialize-insecure
mysqld_safe --defaults-file=/home/opc/archive/8.0/my.cnf &

mysql -uroot -h127.0.0.1 -P8000 -e "set global local_infile=on"
mysqlsh root@localhost:8000 -- util loadDump /home/opc/archive/mariadb/backup --ignoreVersion
```
Create script for binary log shipping (vi /home/opc/archive/mariadb/log_shipping.sh)
```
#!/bin/bash

inotifywait -m /home/opc/archive/mariadb/db -e create -e moved_to |
    while read dir action file; do
        echo "This is binlog file created = $file" 
	cat /home/opc/archive/mariadb/db/bin.index | grep $file | wc -l
	if [ `cat /home/opc/archive/mariadb/db/bin.index | grep $file | wc -l` -gt 0 ]; then 
         	echo "The new Binlog file '$file' appeared in directory '$dir' via '$action'"
       
 		cat /home/opc/archive/mariadb/db/bin.index | grep $file | sed 's/\//\\\//g' > /tmp/log_shipping.dummy
		x=`cat /tmp/log_shipping.dummy`
		echo "sed -n '$x/{x;p;d;}; x' /home/opc/archive/mariadb/db/bin.index > /tmp/log_shipping_identity" > /tmp/log_shipping_dummy.sh
		chmod u+x /tmp/log_shipping_dummy.sh
		/tmp/log_shipping_dummy.sh
		y=`cat /tmp/log_shipping_identity`
		
		echo "Processing $y"	
        	# reading new binlog
		
		/home/opc/archive/mariadb/mariadb-10.9.2-linux-systemd-x86_64/bin/mysqlbinlog $y --base64-output=DECODE-ROWS -vv | grep -v SET | sed 's/#Q>//g' | grep -v "#" | grep -v ROLLBACK | grep -v "\/" | grep -v "START TRANSACTION" | grep -v "DELIMITER" > /tmp/1.lst
		
		rm /home/opc/archive/mariadb/transactions.sql
		
	        while IFS= read -r line; do if [[ $line = " ("* ]]; then if [[ $line = *")" ]]; then echo "$line;" >> /home/opc/archive/mariadb/transactions.sql; else if [[ $line = *"," ]]; then echo "$line" >> /home/opc/archive/mariadb/transactions.sql; else echo "$line);" >> /home/opc/archive/mariadb/transactions.sql; fi; fi; else if [[ $line = *"," ]]; then echo "$line" >> /home/opc/archive/mariadb/transactions.sql; else echo "$line;" >> /home/opc/archive/mariadb/transactions.sql; fi; fi; done < /tmp/1.lst
	
          	# Apply to MySQL 8.0
	  	. /home/opc/.8030.env
          	mysql -uroot -h127.0.0.1 -P8000 -e "source /home/opc/archive/mariadb/transactions.sql";
    	fi
    done
```
Change mode and run 
```
chmod u+x /home/opc/archive/mariadb/log_shipping.sh

sudo yum install -y inotify-tools-3.14-8.el7.x86_64

/home/opc/archive/mariadb/log_shipping.sh &
```
Test by login to mariadb
```
mysql -uroot -h127.0.0.1 -proot -e "create database repl; create table repl.test (i int); insert into repl.test values (1), (2), (3); flush binary logs;"
mysql -uroot -h127.0.0.1 -P8000 -e "select * from repl.test;"
```
