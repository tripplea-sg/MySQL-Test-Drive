# Encryption and Data Masking
## 1. Install Enterprise Encryption
As root:
```
mysql -uroot -proot -h127.0.0.1

CREATE FUNCTION asymmetric_decrypt RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION asymmetric_derive RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION asymmetric_encrypt RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION asymmetric_sign RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION asymmetric_verify RETURNS INTEGER SONAME 'openssl_udf.so';
CREATE FUNCTION create_asymmetric_priv_key RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION create_asymmetric_pub_key RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION create_dh_parameters RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION create_digest RETURNS STRING SONAME 'openssl_udf.so';
```
## 2. Create database instance 3306 and 3307
```
mysqld --defaults-file=3306.cnf --initialize-insecure
mysqld --defaults-file=3307.cnf --initialize-insecure
mysqld_safe --defaults-file=3306.cnf &
mysqld_safe --defaults-file=3307.cnf &
```
## 3. Configure Instance 3306 and 3307
```
mysqlsh -- dba configure-instance { --host=127.0.0.1 --port=3306 --user=root } --clusterAdmin=gradmin --clusterAdminPassword=grpass --interactive=false --restart=true
mysqlsh -- dba configure-instance { --host=127.0.0.1 --port=3307 --user=root } --clusterAdmin=gradmin --clusterAdminPassword=grpass --interactive=false --restart=true
```
## 4. Create replicaset
```
mysqlsh gradmin:grpass@localhost:3306 -e "dba.createReplicaSet('myRS')"
mysqlsh gradmin:grpass@localhost:3306 -e "var rs = dba.getReplicaSet(); rs.addInstance('gradmin:grpass@localhost:3307')"
```
## 5. Show replicaset Status
```
mysqlsh gradmin:grpass@localhost:3306
var rs = dba.getReplicaSet();
rs.status()
\quit
```
## 6. Setup Router
```
mysqlrouter --bootstrap gradmin:grpass@localhost:3306 --directory router 
router/start.sh
```
## 7. Connect to router port 6446 and install world_x schema
```
mysql -uroot -h127.0.0.1 -e "create user root@'%' identified by 'root'; grant all privileges on *.* to root@'%' with grant option;"
mysql -uroot -h127.0.0.1 -P6446 -proot -e "source /home/opc/script/world_x.sql"
```
## 8. Check if world_x schema is available on 3306 and 3307
```
mysql -uroot -h127.0.0.1 -e "show databases;"
mysql -uroot -h127.0.0.1 -P3307 -e "show databases;"
mysql -uroot -h127.0.0.1 -P6446 -proot -e "show databases;"
mysql -uroot -h127.0.0.1 -P6447 -proot -e "show databases;"
```
## 9. Switch Primary from 3306 to 3307
```
mysql -uroot -h127.0.0.1 -P6446 -proot -e "select @@hostname, @@port"
mysqlsh gradmin:grpass@localhost:3306 --replicaset
rs.setPrimaryInstance('gradmin:grpass@localhost:3307')
\quit
mysql -uroot -h127.0.0.1 -P6446 -proot -e "select @@hostname, @@port"
```
## 10. Clean Up environment
```
router/stop.sh
rm -Rf /home/opc/config/router
mysqladmin -uroot -h127.0.0.1 shutdown
mysqladmin -uroot -h127.0.0.1 -P3307 shutdown
rm -Rf /home/opc/data/3306/*
rm -Rf /home/opc/data/3307/*
```






