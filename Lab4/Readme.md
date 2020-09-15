# MySQL Enterprise Backup
## 1. Full database backup:
```
mkdir -p /home/opc/config/backup
mysqlbackup --user=root --password=root --host=127.0.0.1 --port=3306 --backup-dir=/home/opc/config/backup --with-timestamp backup-and-apply-log
```
## 2. Make data change after backup
```
mysql -uroot -h127.0.0.1 -proot -e "create database temp; create table temp.test (i int); insert into temp.test values (1);"
```
## 3. Taking incremmental backup
```
mysqlbackup --user=root --password=root --host=127.0.0.1 --port=3306 --incremental --incremental-backup-dir=/home/opc/config/backup --with-timestamp --incremental-base=history:last_backup backup
```
## 4. Destroy database 3306
```
mysqladmin -uroot -h127.0.0.1 -proot shutdown
rm -Rf /home/opc/data/3306/*
```
## 5. Restore database 3306
```
ls /home/opc/config/backup/
mysqlbackup --datadir=/home/opc/data/3306 --backup-dir=/home/opc/config/backup/<sub-directory-full-backup> copy-back-and-apply-log
mysqlbackup --datadir=/home/opc/data/3306 --incremental-backup-dir=/home/opc/config/backup/2020-09-15_08-52-08 --incremental copy-back-and-apply-log
mysqld_safe --defaults-file=/home/opc/config/3306.cnf &
```
## 6. Check the data
```
mysql -uroot -h127.0.0.1 -proot -e "show databases"
mysql -uroot -h127.0.0.1 -proot -e "select * from temp.test"
```
