

# 什么是ECK

ECK Operator 全称是 Elastic Cloud on Kubernetes Operator (ECK) 。ECK 以 Operator 方式运行在 Kubernetes 集群上，可以自动部署、配置、编排和管理 Elasticsearch、Kibana、APM Server、Enterprise Search、Beats、Elastic Agent 和 Elastic Maps Server，从而简化这些应用程序的部署和管理。

- Elasticsearch 是 Elasticsearch 集群的核心组件，它提供了数据存储、搜索和分析功能。目前全文搜索引擎的首选。它可以快速地存储、搜索和分析海量数据，DCE 5.0 内置的搜索服务基于 Elasticsearch，能够提供分布式搜索服务，为用户提供结构化、非结构化文本以及基于 AI 向量的多条件检索、统计、报表。完全兼容 Elasticsearch 原生接口。

- Kibana 是一款适用于 Elasticsearch 的数据可视化和管理工具，可以提供实时的直方图、线形图、饼状图和地图。Kibana 同时还包括诸如 Canvas 和 Elastic Maps 等高级应用程序；其中 Canvas 允许用户基于自身数据创建定制的动态信息图表，而 Elastic Maps 则可用来对地理空间数据进行可视化。

- APM Server：APM(Server Application Performance Monitoring)组件是 Elasticsearch 集群中的可选组件，它提供应用程序性能监控功能，可以监控应用程序的错误、请求、事务等。ECK 运算符可以在 Elasticsearch 集群中启用 APM Server，并使用 CRD 来管理配置。在运行APM Server的节点中，日志文件提供对集群活动的信息，并与 Elasticsearch 节点交互。

  https://www.elastic.co/guide/en/apm/guide/8.1/apm-components.html

- Enterprise Search 企业级 ES

- Beats：可选组件,它与Elasticsearch 集群交互。Beats 组件是指用于收集、处理和发送数据到 Elasticsearch 集群的轻量级数据收集器。这些组件包括 Filebeat、Metricbeat、Packetbeat、Auditbeat等，并可以通过 Kubernetes 进行管理。

- Elastic Agent: 是指日志、指标、安全数据的代理，使用 Elastic 代理，您可以在每个主机上使用一个统一代理从任何地方收集所有形式的数据。集安装、配置和扩展于一体。即可以从无法直接部署的远程服务和硬件收集和转发数据至ES。

- Elastic Maps Server：是 Elastic Maps Service 的自我管理版本，以 [Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020) 镜像的形式提供，同时提供 EMS 底图和 EMS 边界。Maps Service为Kibana中的所有地理空间可视化（包括地图应用程序）提供支持，通过提供基本地图块、形状文件和地理数据可视化所必需的关键功能。

简而言之，**Elastic Cloud on Kubernetes（ECK）** 是其中的一种 Kubernetes Operator，方便我们管理 Elastic Stack 家族中的各种组件，例如 Elasticsearch，Kibana，APM，Beats 等等。比如只需要定义一个 Elasticsearch 类型的 CRD 对象，ECK 就可以帮助我们快速实现 Elasticsearch 集群的部署、伸缩、备份、监控等功能。实现 Elasticsearch 集群的高效运行和管理。

# 为什么需要 ES

Elastic Cloud on Kubernetes，可以简化如下关键操作：

- 管理和监控多个 ES 集群
- 自定义 ES 集群配置
- 自动扩缩容
- ES 集群版本升级（通过滚动升级执行安全的配置更改）
- 备份和快照
- 使用 TLS 证书保护集群安全
- 建立具有可用区域意识的热温冷（hot-warm-cold）架构

# ECK Operator 功能

主要功能:

- Elasticsearch, Kibana, APM Server, Enterprise Search, Beats 部署
- TLS 证书管理
- 安全的 Elasticsearch 集群配置和拓扑更改
- 持久卷使用情况
- 自定义节点配置和属性
- 安全设置密钥库更新

# ECK Operator 架构讲解

![image-20230410171822705](/Users/songwenjie/Library/Application Support/typora-user-images/image-20230410171822705.png)

ECK operator 是一个 Kubernetes operator，用于管理 Elastic Cloud on Kubernetes (ECK) 集群。它由多个组件组成，主要包括以下组件：

- CRD：用于定义 ECK 集群的自定义资源，包括 Elasticsearch、Kibana、Apm Server 等。
- Controller：用于监听 CRD 变化，根据变化创建、更新或删除对应的资源。
- Controller Manager：作为集群内部的管理控制中心，负责责集群内的 es、kibana等资源的管理，当某个 es 节点意外宕机时，Controller Manager会及时发现并执行自动化修复流程，确保集群始终处于预期的工作状态。

此外还包含：

- Admission Controller：用于验证 ECK 集群的配置是否正确。
- Webhook：用于向 Kubernetes API Server 注册 Admission Controller。

这些组件之间的交互是通过 Kubernetes API Server 实现的。ECK operator 通过监听 Kubernetes API Server 中的 CRD 变化来管理 ECK 集群。当用户创建、更新或删除 CRD 时，ECK operator 的 Controller 会相应地创建、更新或删除对应的资源。而 Admission Controller 则用于验证这些资源的配置是否正确，保障 ECK 集群的正常运行。

综上所述，ECK operator 的各个组件之间通过 Kubernetes API Server 来交互，保障了 ECK 集群的正常管理和运行。

# DCE5.0 ES 架构

MCamel-Elasticsearch 是第五代产品中精选中间件 Elasticsearch 的产品代号。

- 包含ECK  Operator 和 MCamel-Elasticsearch
- MCamel-Elasticsearch 通过操作 CR 来实现对 Elasticsearch Cluster 的操作；
-  MCamel-Elasticsearch 对外同时提供 gRPC 和 REST API，通过使用 grpc-gateway 实现；

**离线安装额外的设置**

离线安装：https://gitlab.daocloud.cn/ndx/mcamel/mcamel-elasticsearch/-/tree/main/charts/mcamel-elasticsearch

Values 中标记为 Offline 变量的参数，需要在离线环境中替换。

```plaintext
--set elasticsearch_image.registry=${localRepoAddr}/elastic.m.daocloud.io
--set exporter_image.registry=${localRepoAddr}/quay.m.daocloud.io
--set kibana_image.registry=${localRepoAddr}/elastic.m.daocloud.io
```

这些参数会传到 apiserver 里面的 deployment 的 ENV 信息里面。apiserver 启动之后，运行时镜像就会被动态替换。

```yaml
env:
- name: ES_IMG
- name: EXPORTER_IMG
- name: KIBANA_IMG
```

参数说明：

```plaintext
--set ui.image.tag: 指定前端镜像版本
--set ghippo.createCrd: 注册ghippo路由,默认开启
--set insight.serviceMonitor.enabled: 开启监控,默认开启
--set insight.grafanaDashboard.enabled: 开启监控面板,默认开启

# 全局参数
--set global.mcamel.imageTag: 镜像版本
--set global.imageRegistry: 镜像仓库地址
```

# CRD 资源

# Elasticsearch Spec 说明

| 参数                    | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| version                 | 版本                                                         |
| image                   | 镜像                                                         |
| nodeSets                | es集群的节点配置，必填。主要包括节点数量、名称、节点配置（远程集群连接是否开启、节点类型 （master、data）），Volume/存储配置和 podTemplate（容器和initContainers配置、设置虚拟内存等） |
| transport               | 包含 service 和 tls配置；                                    |
| auth                    | 包含Elasticsearch的用户身份验证和授权安全设置。              |
| http                    | 连接信息，包含serivce和tls信息                               |
| secureSettings          | 安全配置，配置选项为 Kubernetes secret                       |
| updateStrategy          | 升级策略                                                     |
| podDisruptionBudget     | podDisruptionBudget配置，配置集群最大不可用/最小可用的pod数量 |
| serviceAccountName      | 需要能远程访问ES集群                                         |
| remoteClusters          | 建立单向连接到远程Elasticsearch集群                          |
| volumeClaimDeletePolicy | 删除pvc的策略，仅支持DeleteOnScaledownOnly和DeleteOnScaledownAndClusterDeletion |
| monitoring              | 收集日志和监控数据                                           |
| revisionHistoryLimit    | RevisionHistoryLimit是要保留以允许在 sts 中回滚的版本数。    |

>ES节点有如下角色：
>
>- master 主节点：主节点负责轻量级集群范围的操作，例如创建或删除索引、跟踪哪些节点是集群的一部分以及决定将哪些分片分配给哪些节点。任何不是仅投票节点的主合格节点都可以通过主选举过程选举成为主节点。
>
>- data 数据节点：数据节点保存包含已编入索引的文档的分片。数据节点处理数据相关操作，如 CRUD、搜索和聚合。
>  这些操作是 I/O 密集型、内存密集型和 CPU 密集型的。监控这些资源并在它们过载时添加更多数据节点非常重要。
>  拥有专用数据节点的主要好处是主角色和数据角色的分离。
>
>  在多层部署架构，您可以使用专门的数据角色分配数据节点到指定等级：**data_content，data_hot，data_warm， data_cold，或data_frozen。** 一个节点可以属于多个层，但具有其中一个专用数据角色的节点不能具有通用data角色。
>
>- data_content 内容数据节点：内容数据节点容纳用户创建的内容。它们支持 CRUD、搜索和聚合等操作。
>
>- data_hot 热点数据节点：热数据节点在进入 Elasticsearch 时存储时间序列数据。热层必须快速读取和写入，并且需要更多的硬件资源（例如 SSD 驱动器）。
>
>- data_warm 暖数据节点：暖数据节点存储不再定期更新但仍在查询的索引。查询量的频率通常低于索引处于热层时的频率。性能较低的硬件通常可用于此层中的节点。
>
>- data_cold 冷数据节点：冷数据节点存储访问频率较低的只读索引。此层使用性能较低的硬件，并且可以利用可搜索的快照索引来最小化所需的资源。
>
>- data_frozen 冻结数据节点：冻结层 专门存储部分安装的索引。我们建议您在冻结层中使用专用节点。
>
>- ingest 摄取节点：预处理操作允许在索引文档前，通过事先定义好的一些列processors和pipeline对数据进行某种转换，默认所有节点都启动ingest，也可以通过配置使某节点仅用于预处理。
>
>- ml 机器学习节点：机器学习节点运行作业并处理机器学习 API 请求。
>
>- remote_cluster_client 远程集群客户端节点：远程集群客户端节点充当跨集群客户端并连接到 远程集群。连接后，您可以使用跨集群搜索来搜索远程集群。您还可以使用跨集群复制在集群之间同步数据。
>
>- transform 转换节点：转换节点运行转换并处理转换 API 请求。
>
>- voting_only 仅投票节点：能参与主节点的投票选举环节，但是自己不能被选举为master。
>
>- coordinating 仅协调节点：该节点专用与接收应用的查询连接、接受搜索请求，但其本身不负责存储数据，本质上，**仅协调节点的行为就像智能负载均衡器。**
>
>https://blog.csdn.net/w1014074794/article/details/119676410
>
>**注意⚠️：**
>一个ES集群中，必须有以下角色：
>
>- master
>- data_content and data_hot
>  OR
>  data
>
>
>
>集群的状态有 Green、Yellow 和 Red 三种，如下所述：https://learnku.com/articles/40400
>
>Green：绿色，健康。所有的主分片和副本分片都可正常工作，集群 100% 健康。
>
>Yellow：黄色，预警。所有的主分片都可以正常工作，但至少有一个副本分片是不能正常工作的。此时集群可以正常工作，但是集群的高可用性在某种程度上被弱化。
>
>Red：红色，集群不可正常使用。集群中至少有一个分片的主分片及它的全部副本分片都不可正常工作。
>
>这时虽然集群的查询操作还可以进行，但是也只能返回部分数据（其他正常分片的数据可以返回），而分配到这个分片上的写入请求将会报错，最终会导致数据的丢失
>

# Kibana Spec 说明

| 参数                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| version              | 版本                                                         |
| image                | 镜像                                                         |
| count                | 副本数                                                       |
| config               | Kibana 配置                                                  |
| http                 | 连接信息，包含serivce和tls信息                               |
| elasticsearchRef     | 指定当前集群的某个es cluster，包含命名空间、名称、使用service配置的连接信息；使用secret指定外部集群信息 |
| enterpriseSearchRef  | 指定当前集群的某个企业级es cluster，包含命名空间、名称、使用service配置的连接信息；使用secret指定外部集群信息 |
| podTemplate          | pod 信息配置如 lable、cpu和内存设置等；                      |
| monitoring           | 收集日志和监控数据                                           |
| serviceAccountName   | 需要能访问ES集群                                             |
| revisionHistoryLimit | RevisionHistoryLimit是要保留以允许在  deploy 中回滚的版本数。 |

# Beat  配置说明

| 参数                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| type                 | 要部署的Beat 的类型 (filebeat、metricbeat、heartbeat、auditbeat、journalbeat、packetbeat等)。 |
| version              | 版本                                                         |
| image                | 镜像                                                         |
| elasticsearchRef     | 指定当前集群的某个es cluster，包含命名空间、名称、使用service配置的连接信息；使用secret指定外部集群信息 |
| kibanaRef            | 指定某个kibana，包含命名空间、名称、连接信息、账户信息       |
| config               | Beat配置信息                                                 |
| configRef            | 指定某个secret作为配置                                       |
| daemonSet            | 指定使用daemonSet去部署，支持podTemplate和updateStrategy配置 |
| deployment           | 指定使用deployment去部署，支持podTemplate，replicas，strategy |
| secureSettings       | 安全配置                                                     |
| monitoring           | 收集日志和监控数据                                           |
| serviceAccountName   | 需要能访问ES集群                                             |
| revisionHistoryLimit | RevisionHistoryLimit是要保留以允许在  Deployment/DaemonSet 中回滚的版本数。 |

# 备份于快照

要为 Kubernetes 上的 Elasticsearch 设置自动快照，您必须：

- 使用 Elasticsearch API 注册快照存储库。
- 通过API或Kibana UI 设置快照生命周期管理策略

1. 创建一个 包含 S3 存储的access_key、secret_key 的secret

2. 将 secret 配置到ES 集群中 secureSettings；

3. 调用 API 创建快照 repository；

4. 调用接口创建快照和备份

   创建 snapshot_1 对 my-index 索引做快照，相关参数含义如下：

   - indices：做快照的索引。
   - wait_for_completion=true：是否等待完成快照后再响应，如果为 true 会等快照完成后才响应。（默认为 false，不等快照完成立即响应）
   - ignore_unavailable: 设置为 true 时，当创建快照时忽略不可用的索引。
   - include_global_state: 设置为 false 时，当某个索引所有的主分片不是全部的都可用时，也继续完成快照。

   ```yaml
   PUT /_snapshot/eck-repository/snapshot_1?wait_for_completion=true
   {
     "indices": "my-index",
     "ignore_unavailable": true,
     "include_global_state": false
   }
   ```

   

5. 创建定时快照

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
   ```


# eck 管理 ES reconclie

![image-20230410173537468](/Users/songwenjie/Library/Application Support/typora-user-images/image-20230410173537468.png)

1. Reconcile 函数在收到 ElasticSearch CR 之后，会先对 CR 进行若干合法性检查，如检查 Operator 对 CR 的控制权，包括是否有暂停标记，是否符合 Operator 的版本限制。通过之后，便会调用 internalReconcile 进一步处理.

2. internalReconcile 函数便是开始侧重于对 ElasticSearch CR 的业务合法性进行检查，通过定义若干 validation，对即将进行后续操作的 CR 进行参数合法性检查。当 ES CR 的合法性检查通过之后，便开始了真正的 Reconcile 逻辑。

3. 把后续 Driver 的操作分为三个部分：
   - Reconcile Kubernetes Resource
   - Reconcile ElasticSearch Cluster Business Config & Resource
   - Reconcile Node Spec

4. 首先是清理不匹配的 Kubernetes 资源，然后检查并创建 ScriptsConfigMap（脚本和相关配置的configmap、init 容器和就绪检查），以及 Service。ElasticSearch 会用到三个 Service，便是在这一步进行创建和矫正的：
   - TransportService：headless Service，es 集群 zen discovery 使用
   - InternalService：集群关联的内部服务，用于 operator 对Elasticsearch集群节点执行请求。
   - ExternalService：对 es 节点的 L4 负载均衡 (根据 ES 中 http 的配置进行 service 创建)
   - ReconcileScriptsConfigMap：因为 ES Cluster 是有状态的，因此有部分启动初始化和停机收尾的工作，Operator 会生成相关的脚本，并通过 ConfigMap 挂载到 Pod 中，并在 Pod 的 Lifecycle hook 中进行执行，Operator 会渲染相关个脚本，如：
     - `readiness-probe-script.sh`
     - `pre-stop-hook-script.sh`
     - `prepare-fs.sh`

5. 在对 K8s 的资源调和完成之后，便开始处理 ES 集群运行需要的其他依赖，比如 CA 和证书，用户和权限的配置文件，seed主机的配置等，这些都会创建好相应的 ConfigMap 或 Secret，并等待启动时注入到 Pod 中。

   除此之外， Operator 还在这里初始化了 Observer，这是一个定期轮询 ES 状态的组件，并缓存了当前 Cluster 的最新状态，也可以变相的实现 Cluster Stat Watch。

   ![image-20230410173908801](/Users/songwenjie/Library/Application Support/typora-user-images/image-20230410173908801.png)

   当这些启动依赖准备完成，之后，剩下的就是创建具体的资源尝试把 Pod 拉起来了。

6. 下面分两个阶段进行：正式创建 K8S 资源 sts、pvc 等，当 ES Cluster 就绪后 （通过 Service 是否可以访问 ES 集群），调和ES 资源。

   第一个阶段首先进行 部署前安全检查（检查实际的statefulset和相应的pod是否符合我们的预期）：

   1. 本地的资源对象缓存是否符合期望
   2. StatefulSet 以及 Pod 是否正常（Generation 和 Pod 的数目）

   然后根据 CR 构建期望的 StatefulSet & Service 资源，然后根据 CR 构建期望的 StatefulSet & Service 资源，最后就是用预期的 sts 资源并扩展及若发现有 pending 的pod 或如果存在旧的 Pod 需要更新，便会通过简单有效的 deletepo 将 Pod 删掉，进行强制更新则强制更新；到此第一阶段就进行完了，而相关的 K8s 资源也基本创建完成。

   

   第二阶段：当 Operator 可以通过 http client 访问到 ES 集群后，进行第二阶段的创建工作。首先是基于当前的 Master 数调整 Zen Discovery 配置，以及 Voting 相关配置。后面会再进行 scale down 以及 rolling upgrade 的相关操作，不过对于集群的创建来说，到这里已经完成了。

7. Rolling Upgrades

   因为 ElasticSearch 是一个类似[数据库](https://cloud.tencent.com/solution/database?from=20065&from_column=20065)的有状态应用，因此我对 ES 集群的升级和后续生命周期维护比较感兴趣，在 Reconcile Node Specs 中，Scale Up 做的比较简单，得益于 ES 通过 Zen 实现了基于[域名](https://cloud.tencent.com/act/pro/domain-sales?from=20065&from_column=20065)的自我发现，因此新 Pod 被加入到 Endpoints 后便会自动加入集群中。

   但是因为每个节点都维护了部分 shard，所以节点下线或者节点升级，都会涉及到 shard 数据的处理问题。

   **Scale Down** 或者说下线节点的逻辑并不复杂，依然是计算期望与当前的差异。确定 StatefuleSet 应该把 replica 调整到什么数量。

   如果 replica 为零，直接将 StatefulSet 删除，如果不为零，则开始了节点下线的操作。

   ![img](https://ask.qcloudimg.com/http-save/yehe-1952081/08ofa2n0nq.jpeg?imageView2/2/w/2560/h/7000)

   下线是分两步，首先是计算哪些 Node 需要下线，然后通过 setting api，触发 shard 的重分配，把即将下线的 Node 排除在外。

   最后，会检查 Node 中的 shard 是否被清空，如果没有，则会 requeue 等待下次处理，如果已经被清空则开始执行真正的更新 replica 操作。

   在 Rolling Upgrades 时也是类似的操作，首先是计算新旧资源，并把旧的资源清除，在清除完成后，通过 ES Client 打开 ShardsAllocation，以确保 Cluster 中 Shard 的恢复。

  **ElasticSearch 作为有状态的应用，ElasticSearch Operator 除了管理 K8s 资源外，还利用 ES Client，通过保姆式服务完成了生命周期管理。这样的设计虽然巧妙，但是非常依赖 ES Cluster 自身已有的比较完善的自管理能力（比如数据分片的重调度，自发现等）。**

https://cloud.tencent.com/developer/article/1804678

# Hot-Warm-Cold 架构

在日志管理或面对海量的数据时，为了保证 Elasticsearch 的读写性能，官方建议使用 SSD 固态硬盘。然而面对海量的数据，如果全部使用 SSD 硬盘来存储数据将需要很大的成本。并且有些数据是有时效性的，例如热点新闻，日志等等，对于这些数据我们可能只关心最近一段时间的数据，如果把所有的数据都存储在 SSD 硬盘中将造成存储空间的浪费。

为了解决上述问题，我们可以采用 Hot-Warm-Cold 冷热分离的架构来部署 Elasticsearch 集群。 

- 集群节点层面支持规划节点类型，可以使用性能好、读写快的节点作为 Hot 节点；使用性能相对差些的大容量节点作为 Warm 节点；使用廉价的存储节点作为 Cold 节点，存储时间较早的冷数据。 
- Index Lifecycle Management（索引生命周期管理，简称 ILM) 允许您定义何时在两个阶段之间移动，以及在进入那个阶段时如何处理索引。可以通过ILM 根据时间自动将索引迁移到相应的节点上。

为此Elasticsearch 6.6.0及以上版本提供了[索引生命周期管理ILM](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/index-lifecycle-management.html)功能，将索引生命周期分为4个阶段：hot、warm、cold、delete。其中hot阶段主要负责对索引进行滚动更新操作，warm、cold、delete阶段主要负责进一步处理索引数据，详细说明如下。

| **阶段** | **描述**                                                     |
| -------- | ------------------------------------------------------------ |
| hot      | 热数据阶段。主要处理时序数据的实时写入，可根据索引的文档数、大小、时长决定是否调用rollover API来滚动更新索引。 |
| warm     | warm阶段。索引不再写入，主要用来提供查询。                   |
| cold     | 冷数据阶段。索引不再更新，查询很少，查询速度会变慢。         |
| delete   | 删除数据阶段。索引将被删除。                                 |

>说明：
>
>热阶段：
>
>这个 ILM 策略首先会将[索引优先级](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/recovery-prioritization.html)设置为一个较高的值，以便热索引在其他索引之前恢复。30 天后或达到 50GB 时（符合任何一个即可），该索引将滚动更新，系统将创建一个新索引。该新索引将重新启动策略，而当前的索引（刚刚滚动更新的索引）将在滚动更新后等待 7 天再进入温阶段。
>
>温阶段：
>索引进入温阶段后，ILM 会将索引[收缩](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/_actions.html#ilm-shrink-action)到 1 个分片，将索引[强制合并](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/_actions.html#ilm-forcemerge-action)为 1 个段，并将[索引优先级设置](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/_actions.html#ilm-set-priority-action)为比热阶段低（但比冷阶段高）的值，通过[分配](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/_actions.html#ilm-allocate-action)操作将索引移动到温节点。完成该操作后，索引将等待 30 天（从滚动更新时算起）后进入冷阶段。
>
>冷阶段：
>
>索引进入冷阶段后，ILM 将再次降低[索引优先级](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/_actions.html#ilm-set-priority-action)，以确保热索引和温索引得到先行恢复。然后，ILM 将[冻结](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/_actions.html#ilm-freeze-action)索引并将其移动到冷节点。完成该操作后，索引将等待 60 天（从滚动更新时算起）后进入删除阶段。
>
>删除：
>
>我们还没有讨论过这个删除阶段。简单来说，删除阶段具有用于删除索引的删除操作。在删除阶段，您将始终需要有一个 `min_age` 条件，以允许索引在给定时段内待在热、温或冷阶段。



Hot-Warm-Cold 架构 实验：

1. **部署 es 、kibana 和 Cerebro**

   在 Cerebro 上可以很直观地看到索引在每个节点的分布情况。由于通过 ECK 安装的 Elasticsearch 默认都使用了自签名证书来进行 HTTPS 加密，而  Cerebro 默认只能连接受信任的 HTTPS  或者 HTTP 站点，因此我们在定义 Elasticsearch 资源文件时选择禁用 HTTPS，让 Cerebro 通过 HTTP 访问 Elasticsearch 集群。

   > 注意：ECK 在创建 Elasticsearch 集群时，会根据我们指定的属性为节点添加属性，例如 node.attr.data: hot，我们后续就可以通过 node.attr.data 来区分冷热节点。属性名是可以自己定义的，比如还可以叫 node.attr.my-data。

2. **设置 ILM 策略，设置索引模板**

   Elasticsearch 默认的 ILM 刷新时间为 10 分钟，为了便于我们实验演示，将刷新时间改为 10s。

3. 定义 ILM 策略，其中包含 4 个生命周期阶段：

   - Hot 阶段：将索引优先级设置为一个较高的值 100，以便节点在恢复运行后（例如重启）在 Hot 阶段的索引能够在 Warm 和 Cold 阶段的索引之前恢复。当索引主分片文档数达到 5 个或者索引主分片总容量达到 5mb 时，rollover 滚动更新索引。
   -  Warm 阶段：20s 后索引进入 Warm 阶段，将索引收缩到 1 个主分片，强制合并为 1 个段，并将索引的优先级调低为 50，然后通过 allocate 操作将索引移动到 Warm 节点。(暂无)
   - Cold 阶段：40s 后索引是进入 Cold 阶段，将索引的优先级调为最低值 0，并设置索引为只读状态，然后通过 allocate 操作将索引移动到 Cold 节点。
   - Delete 阶段：60s 后删除该索引。

4. 定义一个索引模板，所有通配符匹配 hot-cold-index* 的索引，都会应用该模板的规则。该索引模板调用了刚刚创建的 ILM 策略，并且将新索引分配到 Hot 节点上，主分片数设置为 3，副本分片为 1。当索引触发 rollover 滚动更新新的索引时，以hot-cold-index 作为别名。

5. 为 hot-cold-index 别名创建第一个索引 hot-cold-index -000001，后续在 rollover 滚动更新索引时，索引名会根据最后的序号递增，例如 hot-cold-index -000002、hot-cold-index-000003 ...。指定第一个索引分配到 Hot 节点上，索引别名为hot-cold-index 。一个别名中的所有索引只能有一个索引可以写入数据，"is_write_index": true 自动设置最新滚动更新的索引可写。

   <img src="/Users/songwenjie/Library/Application Support/typora-user-images/image-20230410175924487.png" alt="image-20230410175924487" style="zoom:50%;" />

6. 往索引中插入 5 条数据。

7. 观察索引迁移
      a. 接下来在 Cerebro 上观察索引的迁移过程，点击页面右上角将刷新时间改为 5s。可以看到在插入 5 条文档后触发了索引的 rollover 操作，新创建了索引 hot-cold-index-000002，现在两个索引都在 Hot 节点上。

      ![img](https://chengzw258.oss-cn-beijing.aliyuncs.com/Article/20210905114609.png)

      

      b. 20s 后，hot-warm-cold-000001 索引被迁移到了 Warm 节点上，主分片的数量降为 1。

      ![img](https://chengzw258.oss-cn-beijing.aliyuncs.com/Article/20210905114727.png)

      c. 40s 后，hot-cold-index-000001 索引被迁移到了 Cold 节点上，此时索引变为只读状态。

      ![img](https://chengzw258.oss-cn-beijing.aliyuncs.com/Article/20210905114800.png)

      d. 60s 后，hot-cold-index-000001 索引被删除。

      ![img](https://chengzw258.oss-cn-beijing.aliyuncs.com/Article/20210813000136.png)



