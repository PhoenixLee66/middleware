#### 1. 操作系统配置

###### 1. 文件描述符

> 在建索引过程中会打开很多小文件

永久设置可修改/etc/security/limits.conf文件

```
elasticsearch  -  nofile  65535
```

###### 2. 禁止交换空间

> Elasticsearch的堆内存可能会被挤到磁盘中，垃圾回收速度会从毫秒级别变成分钟级别，导致节点的响应速度慢甚至和集群断开连接

* 禁止所有交换空间

永久关闭需要修改/etc/fstab 文件

* 配置swappiness

修改 /etc/sysctl.conf文件，设置vm.swappiness = 1，可以使Linux在一般情况下不使用交换，除非万不得已。

* 使用内存锁

使用内存锁可以在ES启动时，锁住一段堆内存，保证堆内存不被挤到磁盘中，对应Linux中的mlockall 系统调用，在ES中配置config/elasticsearch.yml 文件

```
bootstrap.memory_lock: true
```

###### 3. 虚拟内存

> Elasticsearch通过文件映射(mmap)来读取磁盘中的文件，这样可以比read系统调用少一次内存拷贝，也被称为零拷贝

修改/etc/sysctl.conf文件

```
vm.max_map_count=262144
```

###### 4. 设置线程

> Elasticsearch中不同的操作有不同的线程池

修改/etc/security/limits.conf

#### 2. es 配置

###### 1. path.data 和 path.log

> 这两个配置的目录分别用来存放索引数据和日志，它们的默认路径位于$_ES_HOME的子文件夹内

另外path.data支持配置多个目录，每个目录都会用来存放数据，但是单个分片会存放在同一个目录内

###### 2. path.repo

> 配置快照存储路径，用于数据备份与恢复，一般是共享文件存储介质

###### 3. 集群名称

> cluster.name： 每个节点需要配置相同的集群名才能加入同一个集群中，且每个节点只能加入一个集群

###### 3. 节点名称

> node.name

###### 4. 网络地址

> network.host：用于被其他节点发现的本节点IP

#### 3. 服务发现和集群形成设置

##### 1. 版本 6.8.6

###### 1. discovery.zen.ping.unicast.hosts

> 基于单播发现配置种子节点(seed nodes)列表，并根据节点设置的角色来参与主节点的选举

###### 2. discovery.zen.hosts_provider

> 基于文件发现配置种子节点(seed nodes)列表，并根据节点设置的角色来参与主节点的选举

###### 3. discovery.zen.minimum_master_nodes

> 设置了最少有多少个备选主节点参加选举，同时也设置了一个主节点需要控制最少多少个备选主节点才能继续保持主节点身份，一般设置 为备选主节点数/2+1

##### 2. 版本 7.6.2

###### 1. discovery.seed_hosts

> 服务发现主机名，并加入到相同的集群中

###### 2. cluster.initial_master_nodes

> 当开启一个全新的集群时，会有一个集群的引导步骤，用来确定哪些节点（node.name）参与主节点的选举。一般由master节点参与，数据节点不参与。

#### 4. 配置样例

###### 1. 版本 6.8.6

```yaml
cluster.name: imapp-es

node.name: node-1
node.master: false
node.data: true
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
path.repo: ["/usr/share/elasticsearch/backup"]
bootstrap.memory_lock: true
network.host: prod-im-es-01
http.port: 9200
discovery.zen.ping.unicast.hosts: ['prod-im-es-01', 'prod-im-es-02', 'prod-im-es-03', 'prod-im-es-04']
xpack.ml.enabled: false
```

###### 2. 版本 7.6.2

```yaml
cluster.name: action-es

node.name: node-1
node.master: false
node.data: true
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
path.repo: ["/usr/share/elasticsearch/backup"]
bootstrap.memory_lock: true
network.host: prod-user-es-01
http.port: 9200
discovery.seed_hosts: ['prod-user-es-04', 'prod-user-es-05', 'prod-user-es-06']
cluster.initial_master_nodes: ['node-4', 'node-5', 'node-6']
```



<h6>参考资料</h6>

1. [Elasticsearch—生产环境集群核心配置](https://segmentfault.com/a/1190000019900040)


