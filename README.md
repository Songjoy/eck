# Elastic Stack 实战手册 - 3.4.1.8 ECK 实验



创建一个定时快照：

- 我们想在每天的凌晨 1:30 做快照，由于 es 是以 UTC 时间为准，中国的时区是 UTC + 8，倒推 8 小时，因此这里设置时间为每天 17:30。
- 当创建快照时忽略不可用的索引。
- 快照保留 30 天。

```yaml
PUT _slm/policy/daily-snapshot-policy
{
  "name": "<daily-snapshot-policy-{now/d}>",
  "schedule": "0 30 17 * * ?",
  "repository": "eck-repository",
  "config": {
    "ignore_unavailable": true,
    "partial": true
  },
  "retention": {
    "expire_after": "30d"
  }
}
```
























