# MySQL Basic Installation

## 1. Install MySQL Binaries using TAR
Install using TAR is flexible and stright forward. However, we have to ensure all pre-requisites are installed on. new server\
For new server, RPM or YUM based installation can be use to validate any missing OS dependencies.
```
cd /home/opc/bin/tar
tar -zxvf mysql-commercial-8.0.21-el7-x86_64.tar.gz
tar -zxvf mysql-shell-commercial-8.0.21-linux-glibc2.12-x86-64bit.tar.gz
tar -zxvf mysql-router-commercial-8.0.21-el7-x86_64.tar.gz
cd /home/opc/config
```

## 2. Create database with random root password (using "--initialize") and start database
Source environment: a process that we need to include MySQL binary into $PATH
```
. 8021.env
```
See Option File 3306.cnf and examine the content.
```
cat 3306.cnf
```
Create database with random root password using the following:
```
mysqld --defaults-file=3306.cnf --initialize
```
Get random temporary password from server log (see option file 3306.cnf, and look for variable "log-error")
```
cat /home/opc/data/3306/mysqld.log
```
Look at the temporary password written on the last line of the above output, and start database
```
mysqld --defaults-file=3306.cnf &
```

## 3. Login to database as root@'localhost' using temporary password and change the password
Syntax: mysql -u<user> -h<ip/hostname> -p<password> -P<port> \
If port is 3306 (standard), it does not mandatory to use -P3306
```
mysql -uroot -h127.0.0.1 -p
```
Copy and paste the temporary password to login, and change password
```
alter user root@'localhost' identified by 'root';
exit;
```

## 4. Login to database as root@'localhost' using the new password ("root")
```
mysql -uroot -h127.0.0.1 -proot
```
shutdown and exit from MySQL session
```
shutdown;
exit;
```

## 5. Create database with empty root password and change password after login
By using this way, it does not required to view the temporary root password from server log for first time login using root@localhost
### 5.1. Remove all database files on the datadir
```
rm -Rf /home/opc/data/3306/*
```
### 5.2. Create database with empty root password (using "--initialize-insecure") and login to change password
```
mysqld --defaults-file=3306.cnf --initialize-insecure
mysqld --defaults-file=3306.cnf &
```
Now database is up, and we can login. Please observe that we shall not use "-p" anymore because root password is empty.
```
mysql -uroot -h127.0.0.1 
```
Change password of root@'localhost' to 'root'
```
alter user root@'localhost' identified by 'root';
```
Shutdown and exit mysql
```
shutdown;
exit;
```
### 5.3. Start database using mysqld_safe
MySQL has 1 server process and if the server process get killed then database will be down. To provide autostart in Linux, we can register mysql database into linux systemd, and hence it can be started up automatically if the service is enabled. Another way to provide monitoring and autostart to mysql server process is using mysqld_safe. \

Start database using mysqld_safe:
```
mysqld_safe --defaults-file=3306.cnf &
```
Try to connect as root with password 'root'
```
mysql -uroot -h127.0.0.1 -proot
```
Show databases (it will show all default databases installed on the instance) and exit
```
show databases;
exit;
```
Now we try to kill mysql server process from OS level. First, get the Linux PID:
```
ps -ef | grep mysqld | grep -v mysqld_safe
```
The output will show something like the following:
```
opc      xxxx yyyy  0 17:29 pts/0    00:00:00 /home/opc/bin/tar/mysql-commercial-8.0.21-el7-x86_64/bin/mysqld --defaults-file=3306.cnf --basedir=/home/opc/bin/tar/mysql-commercial-8.0.21-el7-x86_64 --datadir=/home/opc/data/3306 --plugin-dir=/home/opc/bin/tar/mysql-commercial-8.0.21-el7-x86_64/lib/plugin --log-error=/home/opc/data/3306/mysqld.log --pid-file=test-drive-preparation.pid --socket=/home/opc/data/3306/mysqld.sock --port=3306
```
Then, kill the above process by replacing "xxxx" with actual PID number:
```
kill -9 xxxx
```
Then, we will see mysqld process is automatically being restarted by mysqld_safe. Now, let's login again:
```
mysql -uroot -h127.0.0.1 -proot
```
By using mysqld_safe to start database, we can also restart database within mysql cli.
```
show databases;
restart;
show databases;
show databases;
exit;
```
## 6. Show database variables and other stuff
Login to mysql
```
mysql -uroot -h127.0.0.1 -proot
```
### 6.1. check status:
```
status;
```
### 6.2. Show status of InnoDB Storage Engine:
```
show engine innodb status;
```
### 6.3. Show all session variables:
```
show variables;
```
Show specific variables (e.g. a variable called GTID_MODE or all variables contains the word "GTID"):
```
show variables like 'gtid_mode';
show variables like '%gtid%';
show variables like '%datadir%';
show variables like 'innodb_file_per_table';
show variables like 'innodb_flush_log_at_trx_commit';
show variables like 'transaction_isolation';
```
How to set / change variables ? \
Set transaction isolation from default "REPEATABLE-READ" into "READ-COMMITTED" (use "persist" for making the change persisted (or no change after restart).
```
set persist transaction_isolation='READ-COMMITTED';
show variables like 'transaction_isolation';
```
### 6.4. Query mysql tablespaces and datafiles
```
select * from information_schema.innodb_tablespaces;
SELECT FILE_ID, FILE_NAME, FILE_TYPE, TABLESPACE_NAME, FREE_EXTENTS, TOTAL_EXTENTS,  EXTENT_SIZE, INITIAL_SIZE, MAXIMUM_SIZE, AUTOEXTEND_SIZE, DATA_FREE, STATUS ENGINE FROM INFORMATION_SCHEMA.FILES;
```
### 6.5. Sys.Diagnostics
Creates a report of the current server status for diagnostic purposes. \
Create a diagnostics report that starts an iteration every 30 seconds and runs for at most 120 seconds using the current Performance Schema settings.
```
tee diag.out;
CALL sys.diagnostics(120, 30, 'current');
notee;
exit;
```
View file diag.out:
```
cat diag.out
```
## 7. Using Index and Histogram
Login to mysql
```
mysql -uroot -h127.0.0.1 -proot
```
Run sql script to install database schema and insert data
```
source /home/opc/script/world_x.sql;
show databases;
use world_x;
```
Column ID is the primary key, therefore the following query will be fast because it does index range scan
```
explain format=tree select * from city where id>4055;
```
Compare to this one which is not using index (look at the number of rows examined)
```
explain format=tree select * from city where district='Florida';
```
If creating index against column district is not an option, try to use histogram to reduce number of rows examined
```
analyze table city update histogram on district with 1000 buckets;
explain format=tree select * from city where district='Florida';
```
## 8. Provision new database using Clone Plugin
Install clone plugin
```
install plugin clone soname 'mysql_clone.so';
create user clone@'%' identified by 'clone';
grant backup_admin on *.* to clone@'%';
show databases;
exit;
```
Create, start database, and login to instance 3307
```
mysqld --defaults-file=3307.cnf --initialize-insecure
mysqld_safe --defaults-file=3307.cnf &
mysql -uroot -h127.0.0.1 -P3307
```
Install clone plugin into instance 3307 and start cloning procedure
```
show databases;
select * from world_x.city;
install plugin clone soname 'mysql_clone.so';
set global valid_clone_donor_list='127.0.0.1:3306'
clone instance from clone@'127.0.0.1':3306 identified by 'clone';
show databases;
show databases;
select * from world_x.city;
```
shutdown and clean-up 3307
```
shutdown;
exit;
rm -Rf /home/opc/data/3307/*
```










