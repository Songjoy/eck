1. 在每个集群上安装部署 ES 
2. 分别在每个集群部署SVC
3. 分别在每个集群部署es，es配置 `discovery.seed_hosts`
 ```yaml
   - config:
      node.roles:
      - data
      - ingest
      - ml
      discovery.seed_hosts:
      - 10.29.16.22:30006
      - 10.29.16.22:30002
    count: 2
    name: data
 ```