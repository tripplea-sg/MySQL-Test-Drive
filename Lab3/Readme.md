# MySQL Enterprise Audit
## 1. Install Audit Plugin and shutdown database
```
mysql -uroot -proot -h127.0.0.1 < /home/opc/bin/tar/mysql-commercial-8.0.21-el7-x86_64/share/audit_log_filter_linux_install.sql
mysqladmin -uroot -proot -h127.0.0.1 shutdown
```
## 2. Look at audit.cnf option file
```
cat audit.cnf
```
audit_log_format=json <-- output format is JSON \
audit_log=FORCE_PLUS_PERMANENT <-- Instance shall run with audit plugin enabled 
## 3. Start database using audit.cnf and login into database
```
mysqld_safe --defaults-file=audit.cnf &
mysql -uroot -h127.0.0.1 -proot
```
## 4. Check audit log plugin 
```
show plugins;
show variables like '%audit_log%';
select * from mysql.audit_log_user;
select * from mysql.audit_log_filter;
```
## 5. Install audit log filter to audit all activities and apply to all users
```
SELECT audit_log_filter_set_filter('log_all', '{ "filter": { "log": true } }');
SELECT audit_log_filter_set_user('%', 'log_all');
select * from mysql.audit_log_user;
select * from mysql.audit_log_filter;
exit;
```
Check content of audit log file
```
cat /home/opc/data/3306/audit.log
```
## 6. Login to database and create test database
Login to database:
```
mysql -uroot -h127.0.0.1 -proot
```
Run the following SQL statements:
```
create database test;
```
Check audit log from mysql session (without logout from mysql):
```
select audit_log_read_bookmark();
select audit_log_read(audit_log_read_bookmark());
exit;
```
Check audit log:
```
cat /home/opc/data/3306/audit.log
```
## 7. Manual Rotate the audit log
Rename existing audit log and rotate audit log
```
mv /home/opc/data/3306/audit.log /home/opc/data/3306/audit.log.bak
ls /home/opc/data/3306/audit.log*
mysql -uroot -h127.0.0.1 -proot -e "set global audit_log_flush=ON;"
ls /home/opc/data/3306/audit.log*
```

