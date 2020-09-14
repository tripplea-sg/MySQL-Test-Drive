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
By using this way, it does not required to view the temporary root password from server log for first time login using root@localhost \
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


