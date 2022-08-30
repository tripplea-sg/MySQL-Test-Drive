# InnoDB ClusterSet
### 1. Create ClusterSet on 3307
```
mysqlsh gradmin:grpass@localhost:3307 -- cluster create-cluster-set clusterset

mysqlsh gradmin:grpass@localhost:3307 -- clusterset status

mysqlsh gradmin:grpass@localhost:3307 -- clusterset status --extended=3
```
### 2. Create Replica Cluster on 5306
Stop replication and delete replication channel1
```
mysql -uroot -proot -h127.0.0.1 -P5306 -e "stop replica for channel 'channel1'; reset replica all for channel 'channel1';"
```
Create Replica Cluster 
```
mysql -uroot -proot -h127.0.0.1 -P5306 -e "set global super_read_only=off;"

mysqlsh -- dba configure-instance { --host=127.0.0.1 --port=5306 --user=root --instance-password=root } --clusterAdmin=gradmin --interactive=false --restart=true

mysqlsh gradmin:grpass@localhost:3307 -- clusterset createReplicaCluster gradmin:grpass@localhost:5306 cluster2 --recoveryMethod=incremental

mysqlsh gradmin:grpass@localhost:3307 -- clusterset status
```
### 3. Adding node to Replica Cluster
```
mysql -uroot -proot -h127.0.0.1 -P5307 -e "set global super_read_only=off;"

mysqlsh -- dba configure-instance { --host=127.0.0.1 --port=5307 --user=root --instance-password=root } --clusterAdmin=gradmin --interactive=false --restart=true

mysqlsh gradmin:grpass@localhost:5306 -- cluster add-instance gradmin:grpass@localhost:5307 --recoveryMethod=incremental

mysqlsh gradmin:grpass@localhost:5307 -- clusterset status
```
### 4. Making cluster2 as PRIMARY cluster
```
mysql -ugradmin -pgrpass -h127.0.0.1 -P6446 -e "select @@port";

mysqlsh gradmin:grpass@localhost:5307 -- clusterset setPrimaryCluster cluster2

mysqlsh gradmin:grpass@localhost:3307 -- clusterset status

mysql -ugradmin -pgrpass -h127.0.0.1 -P6446 -e "select @@port";

mysqlsh gradmin:grpass@localhost:3307 -- cluster status
```
Fix the router
```
/home/opc/software/router/stop.sh

ps -ef | grep mysqlrouter

rm -Rf /home/opc/software/router


```
