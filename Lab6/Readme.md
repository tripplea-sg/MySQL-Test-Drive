# MySQL InnoDB Cluster
Shutdown any running databases (if applicable):
```
mysqladmin -uroot -h127.0.0.1 -P3306 shutdown
mysqladmin -uroot -h127.0.0.1 -P3307 shutdown
mysqladmin -uroot -h127.0.0.1 -P3308 shutdown
```
## 1. Prepare the instance
```
rm -Rf /home/opc/data/3306/*
rm -Rf /home/opc/data/3307/*
rm -Rf /home/opc/data/3308/*
```
## 2. Create database instance 3306, 3307, and 3308
```
cd /home/opc/config
mysqld --defaults-file=3306.cnf --initialize-insecure
mysqld --defaults-file=3307.cnf --initialize-insecure
mysqld --defaults-file=3308.cnf --initialize-insecure
mysqld_safe --defaults-file=3306.cnf &
mysqld_safe --defaults-file=3307.cnf &
mysqld_safe --defaults-file=3308.cnf &
```
## 3. Configure Instance 3306, 3307, and 3308
```
mysqlsh -- dba configure-instance { --host=127.0.0.1 --port=3306 --user=root } --clusterAdmin=gradmin --clusterAdminPassword=grpass --interactive=false --restart=true
mysqlsh -- dba configure-instance { --host=127.0.0.1 --port=3307 --user=root } --clusterAdmin=gradmin --clusterAdminPassword=grpass --interactive=false --restart=true
mysqlsh -- dba configure-instance { --host=127.0.0.1 --port=3308 --user=root } --clusterAdmin=gradmin --clusterAdminPassword=grpass --interactive=false --restart=true
```
## 4. Create Cluster
```
mysqlsh gradmin:grpass@localhost:3306 -- dba createCluster 'myCluster'
mysqlsh gradmin:grpass@localhost:3306 -- cluster status
```
## 5. Add instance 3307 and 3308 to cluster
```
mysqlsh gradmin:grpass@localhost:3306 -- cluster add-instance gradmin:grpass@localhost:3307 --recoveryMethod=clone
mysqlsh gradmin:grpass@localhost:3306 -- cluster add-instance gradmin:grpass@localhost:3308 --recoveryMethod=clone
```
## 6. Setup Router
```
mysqlrouter --bootstrap gradmin:grpass@localhost:3306 --directory router 
router/start.sh
```
## 7.Prepare database users for application to connect
```
mysql -uroot -h127.0.0.1 -e "create user appUser@'%' identified with mysql_native_password by 'appUser'; grant all privileges on *.* to appUser@'%';"
mysql -uroot -h127.0.0.1 -e "create user root@'%' identified with mysql_native_password by ''; grant all privileges on *.* to root@'%' with grant option;"
```
## 8. Disable firewall and launch application
```
sudo systemctl stop firewalld
```
Launch web browser and go to URL "http://your-server-ip/demo"
## 9. Switch Primary from 3306 to 3307
```
mysqlsh gradmin:grpass@localhost:3306 -- cluster setPrimaryInstance gradmin:grpass@localhost:3307
```
## 10. If primary node is crashed
```
ps -ef | grep mysqld | grep 3307 | grep -v mysqld_safe
```
Get the PID and kill that process
```
kill -9 <PID>
```






