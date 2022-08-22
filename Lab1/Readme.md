# MySQL Enterprise Server Installation

## 1. Install MySQL Enterprise Server, MySQL Enterprise Shell and MySQL Enterprise Router using TAR
Install using TAR is flexible and stright forward. However, we have to ensure all pre-requisites are installed on. new server\
For new server, RPM or YUM based installation can be use to validate any missing OS dependencies.
```
cd /home/opc/software

unzip MySQL-Server-8.0.30.zip
tar -zxvf mysql-commercial-8.0.30-el7-x86_64.tar.gz

unzip MySQL-Shell-8.0.30.zip
sudo rpm -ivh mysql-shell-commercial-8.0.30-1.1.el7.x86_64.rpm

unzip MySQL-Router-8.0.30.zip
tar -zxvf mysql-router-commercial-8.0.30-el7-x86_64.tar.gz
```
Create environment file to source MySQL Enterprise. Copy the following line and create $HOME/.8030.env file (command: vi $HOME/.8030.env)
```
PATH=$PATH:/home/opc/software/mysql-commercial-8.0.30-el7-x86_64/bin:/home/opc/software/mysql-router-commercial-8.0.30-el7-x86_64
```

## 2. Create database with random root password (using "--initialize") and start database
Source environment: a process that we need to include MySQL into $PATH
```
. $HOME/.8030.env
```
Create MySQL Database Directory
```
mkdir -p /home/opc/db/3306/data /home/opc/db/3306/innodb_data_home_dir /home/opc/db/3306/innodb_undo_directory /home/opc/db/3306/innodb_temp_tablespace_dir /home/opc/db/3306/innodb_temp_data_file_path /home/opc/db/3306/innodb_log_group_home_dir /home/opc/db/3306/log_bin
```
Create option file / configuration file for the database with the following content (command: vi /home/opc/db/3306/my.cnf)
```
[mysqld]
datadir=/home/opc/db/3306/data
binlog-format=ROW
log-bin=/home/opc/db/3306/log_bin/bin
innodb_data_home_dir=/home/opc/db/3306/innodb_data_home_dir
innodb_undo_directory=/home/opc/db/3306/innodb_undo_directory
innodb_temp_tablespaces_dir=/home/opc/db/3306/innodb_temp_tablespace_dir 
innodb_temp_data_file_path=/home/opc/db/3306/innodb_temp_data_file_path/ibtmp1:12M:autoextend
innodb_log_group_home_dir=/home/opc/db/3306/innodb_log_group_home_dir
port=3306
server_id=10
socket=/home/opc/db/3306/data/mysqld.sock
log-error=/home/opc/db/3306/data/mysqld.log
enforce_gtid_consistency = ON
gtid_mode = ON
log_slave_updates = ON
innodb_buffer_pool_size=12G
innodb_buffer_pool_instances=8
innodb_log_file_size=1G
innodb_log_files_in_group=3
innodb_flush_log_at_trx_commit=1
```
Create database with random root password using the following:
```
mysqld --defaults-file=/home/opc/db/3306/my.cnf --initialize
```
Get random temporary password from server log 
```
cat /home/opc/db/3306/my.cnf | grep log-error
cat /home/opc/db/3306/data/mysqld.log | grep temporary | awk '{print $13}'
```
Start database
```
mysqld --defaults-file=/home/opc/db/3306/my.cnf &
```
Check if database instance is running
```
ps -ef | grep mysqld
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
shutdown;
exit;
```
Start database using mysqld_safe. MySQL has 1 server process and if the server process get killed then database will be down. To provide autostart in Linux, we can register mysql database into linux systemd, and hence it can be started up automatically if the service is enabled. Another way to provide monitoring and autostart to mysql server process is using mysqld_safe. \
```
mysqld_safe --defaults-file=/home/opc/db/3306/my.cnf &  
```  
Check if database instance is running
```
ps -ef | grep mysqld
```
## 4. Login to database as root@'localhost' using the new password ("root")
```
mysql -uroot -h127.0.0.1 -proot
```
Exit from MySQL session
```
mysql > exit;
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
## 5. Show database variables and other stuff
### 5.1. check status:
```
status;
```
### 5.2. Show status of InnoDB Storage Engine:
```
show engine innodb status;
```
### 5.3. Show all session variables:
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
Set transaction isolation from default "REPEATABLE-READ" into "READ-COMMITTED".
```
set persist transaction_isolation='READ-COMMITTED';
restart;
show variables like 'transaction_isolation';
```
### 5.4. Query mysql tablespaces and datafiles
```
select * from information_schema.innodb_tablespaces;
SELECT FILE_ID, FILE_NAME, FILE_TYPE, TABLESPACE_NAME, FREE_EXTENTS, TOTAL_EXTENTS,  EXTENT_SIZE, INITIAL_SIZE, MAXIMUM_SIZE, AUTOEXTEND_SIZE, DATA_FREE, STATUS ENGINE FROM INFORMATION_SCHEMA.FILES;
```
### 5.5. Sys.Diagnostics
Creates a report of the current server status for diagnostic purposes. \
Create a diagnostics report that starts an iteration every 30 seconds and runs for at most 120 seconds using the current Performance Schema settings.
```
tee diag.out;
CALL sys.diagnostics(120, 30, 'current');
notee;
```
View file diag.out:
```
\! cat diag.out
```
## 6. Password Policy
Show default authentication plugin used by the instance:
```
show variables like 'default_authentication_plugin';  
```
Some authentication plugins store account credentials internally to MySQL, in the mysql.user system table: Caching_sha2_password, mysql_native_password, and sha256_password. Caching_sha2_password is default in MySQL 8.0.
### 6.1. Password Expiration Policy
MySQL enables database administrators to expire account passwords manually, and to establish a policy for automatic password expiration. Expiration policy can be established globally, and individual accounts can be set to either defer to the global policy or override the global policy with specific per-account behavior. 
```
create user apps@'%' identified by 'apps';
alter user apps@'%' password expire;
select * from mysql.user where user='apps' \G  
exit;
```
Exit and try to login using apps@'%'
```
mysql -uapps -h127.0.0.1 -papps
```
If the password is expired (whether manually or automatically), the server either disconnects the client or restricts the operations permitted to it.
```
select 1;
  
set password='apps2';
select 1;
  
exit;
```
login using root@'localhost'
```
mysql -uroot -h127.0.0.1 -proot
```
To establish a global policy, default_password_lifetime can also be set and persisted at runtime:
```
SET PERSIST default_password_lifetime = 180;
SET PERSIST default_password_lifetime = 0;  
```
Account based password policy:
```
alter user apps@'%' password expire interval 90 day;

alter user apps@'%' password expire never;
  
alter user apps@'%' password expire default;
```
### 6.2. Password Reuse Policy
MySQL enables restrictions to be placed on reuse of previous passwords. Reuse restrictions can be established based on number of password changes, time elapsed, or both. 
```
SET PERSIST password_history = 6;
SET PERSIST password_reuse_interval = 365;  
```
Account based password reuse policy:
```
alter user apps@'%' password HISTORY 5;

alter user apps@'%' password REUSE INTERVAL 365 DAY;
  
alter user apps@'%' password HISTORY default;
  
alter user apps@'%' password REUSE INTERVAL default;
```
### 6.3. Dual Password
user accounts are permitted to have dual passwords, designated as primary and secondary passwords. Dual-password capability makes it possible to seamlessly perform credential changes in scenarios like this: A system has a large number of MySQL servers, possibly involving replication. Multiple applications connect to different MySQL servers. Periodic credential changes must be made to the account or accounts used by the applications to connect to the servers.
```
alter user apps@'%' identified by 'apps' retain current password;  
```
Exit and do login test using apps (new password) and apps2 (old password)
```
mysql -uapps -h127.0.0.1 -papps2 -e "show databases"
mysql -uapps -h127.0.0.1 -papps -e "show databases"
```
Login as root and discard old password
```
mysql -uroot -h127.0.0.1 -proot -e "alter user apps@'%' discard old password"
  
mysql -uapps -h127.0.0.1 -papps2 -e "show databases"
mysql -uapps -h127.0.0.1 -papps -e "show databases"
```


  
  








