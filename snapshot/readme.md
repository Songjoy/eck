
1. 部署 es 和 kibana
  
2. 访问kibana：https://172.30.120.61:30330/app/dev_tools#/console
   elastic/4gVUu7yIW51a36KI2r26q0QU

3. 准备数据：往 my-index 索引插入几条数据，之后会创建快照备份这个索引。

```
PUT _bulk
{"index":{"_index":"my-index"}}
{"name":"Tom","age":18}
{"index":{"_index":"my-index"}}
{"name":"Jack","age":20}
{"index":{"_index":"my-index"}}
{"name":"Mark","age":21}
```
4. 创建 repository
创建一个 S3 类型的 repository.base_path 是存储中的目录名，创建 repository 的时候会自动在S3 bucket 中创建该目录
GET /_snapshot/my_s3_repository 


POST _snapshot/my_s3_repository
{
  "type": "s3",
  "settings": {
    "bucket": "hackday-redis",  // hackday-redis 必须在 S3 中存在
    "path_style_access": true,	
    "endpoint": "http://minio-api.ats.io",
    "client": "default" 
  }
}
5. 创建快照+备份


PUT /_snapshot/my_s3_repository/es_snapshot_3?wait_for_completion=true
{
  "indices": "my-index",
  "ignore_unavailable": true,
  "include_global_state": false
}

```
GET /_snapshot/my_s3_repository 


POST _snapshot/my_s3_repository
{
  "type": "s3",
  "settings": {
    "bucket": "hackday-redis",
    "path_style_access": true,	
    "endpoint": "http://minio-api.ats.io",
    "client": "default" 
  }
}

PUT /_snapshot/my_s3_repository/es_snapshot_3?wait_for_completion=true
{
  "indices": "my-index",
  "ignore_unavailable": true,
  "include_global_state": false
}
```

6. 恢复

GET my-index/_search

DELETE my-index

POST _snapshot/my_s3_repository/es_snapshot_3/_restore
{
  "indices": "my-index",
  "ignore_unavailable": true,
  "rename_pattern": "(.+)",
  "rename_replacement": "restored_$1"
}

GET restored_my-index/_search

7. 创建定时备份



PUT _slm/policy/daily-snapshot-policy
{
  "name": "<daily-snapshot-policy-{now/d}>",
  "schedule": "0 30 10 * * ?",
  "repository": "my_s3_repository",
  "config": {
    "ignore_unavailable": true,
    "partial": true
  },
  "retention": {
    "expire_after": "30d"
  }
}