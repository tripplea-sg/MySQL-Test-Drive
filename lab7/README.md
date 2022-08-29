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
### 4. Restore to new instance
Create directory
```
mkdir -p /home/opc/db/5306/data /home/opc/db/5306/innodb_data_home_dir /home/opc/db/5306/innodb_undo_directory /home/opc/db/5306/innodb_temp_tablespace_dir /home/opc/db/5306/innodb_temp_data_file_path /home/opc/db/5306/innodb_log_group_home_dir /home/opc/db/5306/log_bin
```
Create option file for this instance (vi /home/opc/db/5306/my.cnf)
```
[mysqld]
datadir=/home/opc/db/5306/data
binlog-format=ROW
log-bin=/home/opc/db/5306/log_bin/bin
innodb_data_home_dir=/home/opc/db/5306/innodb_data_home_dir
innodb_undo_directory=/home/opc/db/5306/innodb_undo_directory
innodb_temp_tablespaces_dir=/home/opc/db/5306/innodb_temp_tablespace_dir 
innodb_temp_data_file_path=/home/opc/db/5306/innodb_temp_data_file_path/ibtmp1:12M:autoextend
innodb_log_group_home_dir=/home/opc/db/5306/innodb_log_group_home_dir
port=5306
server_id=100
socket=/home/opc/db/5306/data/mysqld.sock
log-error=/home/opc/db/5306/data/mysqld.log
enforce_gtid_consistency = ON
gtid_mode = ON
log_slave_updates = ON
innodb_buffer_pool_size=1G
innodb_buffer_pool_instances=1
innodb_log_file_size=1G
innodb_log_files_in_group=3
innodb_flush_log_at_trx_commit=1
```
Restore backup to instance 5306
```
mysqlbackup --defaults-file=/home/opc/db/5306/my.cnf --backup-dir='/home/opc/db/backup/2022-08-29_18-45-16' --encrypt-password=password copy-back-and-apply-log

cp /home/opc/db/backup/2022-08-29_18-45-16/meta/keyring_kef /home/opc/db/5306/data/component_keyring_encrypted_file
```
Start instance 5306
```
mysqld_safe --defaults-file=/home/opc/db/5306/my.cnf &

mysql -uroot -proot -h127.0.0.1 -P5306 --skip-binary-as-hex -e "select * from world_x.city_info_encrypted;"
```
### 5. Build Replication from InnoDB Cluster to standalone
Connect to PRIMARY instance through MySQL Router and create replication user
```
mysql -ugradmin -pgrpass -h127.0.0.1 -P6446 -e "create user repl@'%' identified by 'repl'; grant replication slave on *.* to repl@'%';"
```
connect to INSTANCE 5306 and configure replication channel
```
mysql -uroot -proot -h127.0.0.1 -P5306 -e "CHANGE REPLICATION SOURCE TO SOURCE_HOST='127.0.0.1', SOURCE_USER='repl', SOURCE_PASSWORD='repl', SOURCE_PORT=6447, SOURCE_AUTO_POSITION = 1, master_ssl=1, get_master_public_key=1 for channel 'channel1'; start replica for channel 'channel1'; show replica status for channel 'channel1' \G"

mysql -uroot -proot -h127.0.0.1 -P5306 -e "set persist super_read_only=on;"
```
### 6. Options using MySQL Enterprise Backup
Encrypted and compressed full backup
```
openssl rand -hex 32 > /home/opc/db/backup/meb-key

rm -Rf /var/tmp/backup/*

mysqlbackup --user=mysqlbackup --password=password --host=127.0.0.1 --port=3308 --backup-image=/home/opc/db/backup/ic-backup.enc --encrypt --key-file=/home/opc/db/backup/meb-key  --backup-dir=/var/tmp/backup --read-threads=1 --process-threads=1 --write-threads=1  --compress --compress-level=5 --encrypt-password=password backup-to-image
```
Incremental Backup
```
mysql -uroot -proot -h127.0.0.1 -P3307 -e "create database dev"

rm -Rf /var/tmp/backup/*

mysqlbackup --user=mysqlbackup --password=password --host=127.0.0.1 --port=3308 --backup-image=/home/opc/db/backup/ic-backup-inc.enc --encrypt --key-file=/home/opc/db/backup/meb-key  --backup-dir=/var/tmp/backup --incremental --incremental-base=history:last_backup --encrypt-password=password backup-to-image
```
Create directory for restoration
```
mkdir -p /home/opc/db/5307/data /home/opc/db/5307/innodb_data_home_dir /home/opc/db/5307/innodb_undo_directory /home/opc/db/5307/innodb_temp_tablespace_dir /home/opc/db/5307/innodb_temp_data_file_path /home/opc/db/5307/innodb_log_group_home_dir /home/opc/db/5307/log_bin
```
Create option file for this instance (vi /home/opc/db/5307/my.cnf)
```
[mysqld]
datadir=/home/opc/db/5307/data
binlog-format=ROW
log-bin=/home/opc/db/5307/log_bin/bin
innodb_data_home_dir=/home/opc/db/5307/innodb_data_home_dir
innodb_undo_directory=/home/opc/db/5307/innodb_undo_directory
innodb_temp_tablespaces_dir=/home/opc/db/5307/innodb_temp_tablespace_dir 
innodb_temp_data_file_path=/home/opc/db/5307/innodb_temp_data_file_path/ibtmp1:12M:autoextend
innodb_log_group_home_dir=/home/opc/db/5307/innodb_log_group_home_dir
port=5307
server_id=200
socket=/home/opc/db/5307/data/mysqld.sock
log-error=/home/opc/db/5307/data/mysqld.log
enforce_gtid_consistency = ON
gtid_mode = ON
log_slave_updates = ON
innodb_buffer_pool_size=1G
innodb_buffer_pool_instances=1
innodb_log_file_size=1G
innodb_log_files_in_group=3
innodb_flush_log_at_trx_commit=1
```
Restore backup to instance 5307
```
rm -Rf /var/tmp/backup/*

mysqlbackup --defaults-file=/home/opc/db/5307/my.cnf -uroot --backup-image=/home/opc/db/backup/ic-backup.enc --backup-dir=/var/tmp/backup --datadir=/home/opc/db/5307/data --uncompress --decrypt --key-file=/home/opc/db/backup/meb-key --encrypt-password=password copy-back-and-apply-log

rm -Rf /var/tmp/backup/*

mysqlbackup --defaults-file=/home/opc/db/5307/my.cnf -uroot --backup-image=/home/opc/db/backup/ic-backup-inc.enc --backup-dir=/var/tmp/backup --datadir=/home/opc/db/5307/data --decrypt --key-file=/home/opc/db/backup/meb-key --encrypt-password=password copy-back-and-apply-log

```
Start instance 5307
```
mysqld_safe --defaults-file=/home/opc/db/5307/my.cnf &

mysql -uroot -proot -h127.0.0.1 -P5307 --skip-binary-as-hex -e "show databases"

mysql -uroot -proot -h127.0.0.1 -P5307 --skip-binary-as-hex -e "select * from world_x.city_info_encrypted;"
```
