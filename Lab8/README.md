# InnoDB ClusterSet
### 1. Create ClusterSet on 3307
```
mysqlsh gradmin:grpass@localhost:3307 -- cluster create-cluster-set clusterset

mysqlsh gradmin:grpass@localhost:3307 -- clusterset status

mysqlsh gradmin:grpass@localhost:3307 -- clusterset status --extended=3
```

