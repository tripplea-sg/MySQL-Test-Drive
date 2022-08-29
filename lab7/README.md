# MySQL Enterprise Backup
### 1. Create backup directory
```
mkdir /home/opc/db/backup
```
### 2. Create backup administrator user
```
mysql -uroot -proot -h127.0.0.1 -P3307

CREATE USER mysqlbackup@'%' IDENTIFIED BY 'password';
GRANT SELECT, BACKUP_ADMIN, RELOAD, PROCESS, SUPER, REPLICATION CLIENT ON *.*  TO mysqlbackup@'%';
GRANT CREATE, INSERT, DROP, UPDATE ON mysql.backup_progress TO mysqlbackup@'%'; 
GRANT CREATE, INSERT, DROP, UPDATE, SELECT, ALTER ON mysql.backup_history TO mysqlbackup@'%';
exit
```
### 3. Backup Secondary Node
```
mysqlbackup --user=mysqlbackup --password=password --host=127.0.0.1 --port=3306 --backup-dir=/home/opc/db/backup --with-timestamp --encrypt-password=password backup-and-apply-log

ls /home/opc/db/backup
```

