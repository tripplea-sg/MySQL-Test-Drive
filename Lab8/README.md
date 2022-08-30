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
Create Replica Cluster - we don't need configure-instance because the instance was created from a backup of innodb cluster
```

```
