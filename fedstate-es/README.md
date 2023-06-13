1. 在每个集群上安装部署 ES 
2. 分别在每个集群中，为每个ES 节点 部署transport SVC，且对应ES集群中对应的 `transport.port`, 参考：transport-svc-76.yaml
3. 分别在每个集群部署es，es配置如下
 ```yaml
  - config:
    node.roles:
    - master
    discovery.seed_hosts:
    - 10.29.26.76:30001
    - 10.29.26.78:30003
    cluster.initial_master_nodes:
    - 10.29.26.76:30001
    - 10.29.26.78:30003
    network.publish_host:
    - 10.29.26.76
    transport.port: 30001
 ```
 4. 考虑用户证书，可以先起来一个ES，将 Secret Copy 下来，具体参考 `./confgFile-76` 目录下文件
    
    注意：ES 配置中（config），当定义的节点名称不同时，需要修改 `mes01-es-data-es-transport-certs.yaml` 和  `mes01-es-master-es-transport-certs.yaml` 中的信息。
    
    以上两个文件命名规则 `mes01-es-{Data节点名称}-es-transport-certs.yaml` 和 `mes01-es-{Mater节点名称}-es-transport-certs.yaml`

    ```yaml
    # Data
    apiVersion: v1
    data:
      ca.crt: LS0tLS1CtLS0K
      mes01-es-data-0.tls.crt: LS0tLS1CRUdJTiBDRVJULS0tLS0K  # key 中 mes01-es-data-0 对应每一个节点 pod 名称
      mes01-es-data-0.tls.key: LS0tLS1CRUdJTiBLQo= # key 中 mes01-es-data-0 对应每一个节点 pod 名称
      mes01-es-data-1.tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJS0K # key 中 mes01-es-data-0 对应每一个节点 pod 名称
      mes01-es-data-1.tls.key: FNXZuajZzN # key 中 mes01-es-data-0 对应每一个节点 pod 名称
    kind: Secret
    metadata:
      name: mes01-es-data-es-transport-certs  # mes01-es-{Data节点名称}-es-transport-certs.yaml`=
    type: Opaque

    ---
    # Master
    apiVersion: v1
    data:
      ca.crt: LS0tLS1CRUd0K
      mes01-es-master-0.tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJLS0tCg==  # key 中 mes01-es-master-0 对应每一个节点 pod 名称
      mes01-es-master-0.tls.key: LS0tLS1CRUdJTiBSU0EgUtLQo=   # key 中 mes01-es-master-0 对应每一个节点 pod 名称
    kind: Secret
    metadata:
      name: mes01-es-master-es-transport-certs # mes01-es-{Mater节点名称}-es-transport-certs.yaml
    type: Opaque
    ```