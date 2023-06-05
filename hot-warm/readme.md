# 部署 es 、kibana 和 Cerebro

在 Cerebro 上可以很直观地看到索引在每个节点的分布情况。由于通过 ECK 安装的 Elasticsearch 默认都使用了自签名证书来进行 HTTPS 加密，而  Cerebro 默认只能连接受信任的 HTTPS  或者 HTTP 站点，因此我们在定义 Elasticsearch 资源文件时选择禁用 HTTPS，让 Cerebro 通过 HTTP 访问 Elasticsearch 集群。

# 登录kibana：https://172.30.120.61:31300/ 和 Cerebro： http://172.30.120.61:32084/

elastic/03QdhICZ9C8318VrTV7lnq54

# 设置 ILM 策略，设置索引模板

1. Elasticsearch 默认的 ILM 刷新时间为 10 分钟，为了便于我们实验演示，将刷新时间改为 10s。

   ```json
   PUT _cluster/settings
   {
     "persistent": {
       "indices.lifecycle.poll_interval":"10s"
     }
   }
   ```

   

2. 定义 ILM 策略，其中包含 4 个生命周期阶段：

   - Hot 阶段：将索引优先级设置为一个较高的值 100，以便节点在恢复运行后（例如重启）在 Hot 阶段的索引能够在 Warm 和 Cold 阶段的索引之前恢复。当索引主分片文档数达到 5 个或者索引主分片总容量达到 5mb 时，rollover 滚动更新索引。
   -  Warm 阶段：20s 后索引进入 Warm 阶段，将索引收缩到 1 个主分片，强制合并为 1 个段，并将索引的优先级调低为 50，然后通过 allocate 操作将索引移动到 Warm 节点。(暂无)
   - Cold 阶段：40s 后索引是进入 Cold 阶段，将索引的优先级调为最低值 0，并设置索引为只读状态，然后通过 allocate 操作将索引移动到 Cold 节点。
   - Delete 阶段：60s 后删除该索引。

   ```json
   PUT _ilm/policy/hot-warm-ilm
   {
     "policy": {
       "phases": {
         "hot": {
           "actions": {
             "rollover": {
               "max_size":"5mb",
               "max_docs": "5"
             },
             "set_priority": {
               "priority":100
             }
           }
         },
         "warm": {
           "min_age":"20s",
           "actions": {
             "forcemerge": {
               "max_num_segments":1
             },
             "shrink": {
               "number_of_shards":1
             },
             "allocate": {
               "include": {
                 "data": "warm"
               }
             },
             "set_priority": {
               "priority":50
             }
           }
         },
         "cold": {
           "min_age":"40s",
           "actions": {
             "set_priority": {
               "priority":0
             },
             "readonly": {},
             "allocate": {
               "include": {
                 "data": "cold"
               }
             }
           }
         },
         "delete": {
           "min_age":"60s",
           "actions": {
             "delete": {}
           }
         }
       }
     }
   }
   ```
   
   ```json
   PUT _ilm/policy/hot-cold-ilm
   {
     "policy": {
       "phases": {
         "hot": {
           "actions": {
             "rollover": {
               "max_size":"5mb",
               "max_docs": "5"
             },
             "set_priority": {
               "priority":100
             }
           }
         },
         "cold": {
           "min_age":"40s",
           "actions": {
             "set_priority": {
               "priority":0
             },
             "readonly": {},
             "allocate": {
               "include": {
                 "data": "cold"
               }
             }
           }
         },
         "delete": {
           "min_age":"60s",
           "actions": {
             "delete": {}
           }
         }
       }
     }
   }
   ```

   ECK 在创建 Elasticsearch 集群时，会根据我们指定的属性为节点添加属性，例如 node.attr.data: hot，我们后续就可以通过 node.attr.data 来区分冷热节点。属性名是可以自己定义的，比如还可以叫 node.attr.my-data。



3. 定义一个索引模板，所有通配符匹配 hot-warm-index* 的索引，都会应用该模板的规则。该索引模板调用了刚刚创建的 ILM 策略，并且将新索引分配到 Hot 节点上，主分片数设置为 3，副本分片为 1。当索引触发 rollover 滚动更新新的索引时，以 hot-warm-index 作为别名。

   ```json
   PUT /_template/hot-warm-index-template
   {
     "index_patterns" : [
         "hot-warm-index*"
       ],
       "settings" : {
         "index" : {
           "lifecycle" : {
             "name" : "hot-warm-ilm",
             "rollover_alias" : "hot-warm-index"
           },
           "routing" : {
             "allocation" : {
               "include" : {
                 "data" : "hot"
               }
             }
           },
           "number_of_shards" : 3,
           "number_of_replicas": 1
         }
       }
   }
   ```
   
   ```json
   PUT /_template/hot-warm-index-template
   {
     "index_patterns" : [
         "hot-cold-index*"
       ],
       "settings" : {
         "index" : {
           "lifecycle" : {
             "name" : "hot-cold-ilm",
             "rollover_alias" : "hot-cold-index"
           },
           "routing" : {
             "allocation" : {
               "include" : {
                 "data" : "hot"
               }
             }
           },
           "number_of_shards" : 3,
           "number_of_replicas": 1
         }
       }
   }
   ```

4. 为 hot-warm-index 别名创建第一个索引 hot-warm-index-000001，后续在 rollover 滚动更新索引时，索引名会根据最后的序号递增，例如 hot-warm-index-000002、hot-warm-index-000003、hot-warm-index-000004 ...。指定第一个索引分配到 Hot 节点上，索引别名为 hot-warm-index。一个别名中的所有索引只能有一个索引可以写入数据，"is_write_index": true 自动设置最新滚动更新的索引可写。


   ```json
   PUT hot-warm-index-000001
   {
     "aliases": {
       "hot-warm-index": {
         "is_write_index": true 
       }
     }
   }
   ```
   
   
   ```json
   PUT hot-cold-index-000001
   {
     "aliases": {
       "hot-cold-index": {
         "is_write_index": true 
       }
     }
   }
   ```
5. 往索引中插入 5 条数据。

   ```json
   POST _bulk
   {"index":{"_index":"hot-warm-index"}}
   {"name":"Erlend","age":16}
   {"index":{"_index":"hot-warm-index"}}
   {"name":"Brynjar","age":18}
   {"index":{"_index":"hot-warm-index"}}
   {"name":"Fox","age":18}
   {"index":{"_index":"hot-warm-index"}}
   {"name":"Frank","age":23}
   {"index":{"_index":"hot-warm-index"}}
   {"name":"Sam","age":18}
   ```
   
   ```json
   POST _bulk
   {"index":{"_index":"hot-cold-index"}}
   {"name":"Erlend","age":16}
   {"index":{"_index":"hot-cold-index"}}
   {"name":"Brynjar","age":18}
   {"index":{"_index":"hot-cold-index"}}
   {"name":"Fox","age":18}
   {"index":{"_index":"hot-cold-index"}}
   {"name":"Frank","age":23}
   {"index":{"_index":"hot-cold-index"}}
   {"name":"Sam","age":18}
   ```

6. 观察索引迁移
   a. 接下来在 Cerebro 上观察索引的迁移过程，点击页面右上角将刷新时间改为 5s。可以看到在插入 5 条文档后触发了索引的 rollover 操作，新创建了索引 hot-warm-index-000002，现在两个索引都在 Hot 节点上。

   ![img](https://chengzw258.oss-cn-beijing.aliyuncs.com/Article/20210905114609.png)

   

   b. 20s 后，hot-warm-index-000001 索引被迁移到了 Warm 节点上，主分片的数量降为 1。

   ![img](https://chengzw258.oss-cn-beijing.aliyuncs.com/Article/20210905114727.png)

   c. 40s 后，hot-warm-index-000001 索引被迁移到了 Cold 节点上，此时索引变为只读状态。

   ![img](https://chengzw258.oss-cn-beijing.aliyuncs.com/Article/20210905114800.png)

   d. 60s 后，hot-warm-index-000001 索引被删除。

   ![img](https://chengzw258.oss-cn-beijing.aliyuncs.com/Article/20210813000136.png)
