---
layout: post
title: 2021年04月05日 ElasticSearch 集群安装
date: 2021-04-05
description: "ElasticSearch 集群安装"
tags: ElasticSearch 集群 ES
---   
# ES 集群搭建
#### 装备工作

>下载ElasticSearch安装包（6.3.2）

```sh
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.3.2.tar.gz
```

安装路径

![安装目录结构](https://gitee.com/dbin0123/picgo/raw/master/image/20210405120023.png)

- es_master：主节点
- es_slave_01: 子节点1
- es_slave_02: 子节点2

>解压安装文件至 （es_master,es_slave_01,es_slave_02）

1. es_master安装

>master配置文件（elasticsearch.yml）

```yml

# 集群名称
cluster.name: aiwiown-master
# 节点名称
node.name: master
# 绑定IP
network.host: 127.0.0.1
# http端口
http.port: 9200

# head插件配置跨域
http.cors.enabled: true
http.cors.allow-origin: "*"

```

>启动 ./bin/elasticsearch -d (-d 后台启动)

![head插件查看](https://gitee.com/dbin0123/picgo/raw/master/image/20210405124149.png)

2. es_slave_01安装

>slave_01配置文件（elasticsearch.yml）

```yml

# 集群名称
cluster.name: aiwiown-master
# 节点名称
node.name: slave_01
# 绑定IP
network.host: 127.0.0.1
# http端口
http.port: 9201
# master节点
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
# 为了避免脑裂，集群节点数最少为 半数+1
discovery.zen.minimum_master_nodes: 2

```

>启动 ./bin/elasticsearch -d (-d 后台启动)

2. es_slave_02安装

>slave_02配置文件（elasticsearch.yml）

```yml

# 集群名称
cluster.name: aiwiown-master
# 节点名称
node.name: slave_02
# 绑定IP
network.host: 127.0.0.1
# http端口
http.port: 9202
# master节点
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
# 为了避免脑裂，集群节点数最少为 半数+1
discovery.zen.minimum_master_nodes: 2

```

>启动 ./bin/elasticsearch -d (-d 后台启动)

![head插件查看](https://gitee.com/dbin0123/picgo/raw/master/image/20210405125429.png)

### 附录

#### 配置描述（转：https://www.cnblogs.com/sunsky303/p/9438737.html）

|  配置   | 描述  |
|  ----  | ----  |
| cluster.name  | 配置elasticsearch的集群名称，默认是elasticsearch |
| node.name  | 节点名称（默认随机指定一个name.txt列表中名字） |
| node.master  | 指定该节点是否有<span style='color:red;'>资格</span>被选举成为master，默认是true，elasticsearch默认集群中的第一台启动的机器为master，master挂了会重新选举master |
| node.data  | 指定该节点是否存储索引数据，默认为true，（如果节点配置node.master:false并且node.data: false，则该节点将起到负载均衡的作用） |
| index.number_of_shards  | 设置索引分片个数，默认为5片 |
| index.number_of_replicas  | 设置默认索引副本个数，默认为1个副本（默认5个分片1个拷贝；即总分片数为10） |
| path.conf  | 设置配置文件的存储路径，默认是es根目录下的config文件夹 |
| path.data  | 设置索引数据的存储路径，默认是es根目录下的data文件夹，可以设置多个存储路径，用逗号隔开。 |
| path.work  | 设置临时文件的存储路径，默认是es根目录下的work文件夹。 |
| path.logs  | 设置日志文件的存储路径，默认是es根目录下的logs文件夹。 |
| path.plugins  | 设置插件的存放路径，默认是es根目录下的plugins文件夹。 |
| bootstrap.mlockall  | <span style='color:red;'>设置为true来锁住内存。因为当jvm开始swapping时es的效率会降低，所以要保证它不swap，可以把ES_MIN_MEM和ES_MAX_MEM两个环境变量设置成同一个值，并且保证机器有足够的内存分配给es。同时也要允许elasticsearch的进程可以锁住内存，linux下可以通过ulimit -l unlimited命令。</span> |
| network.bind_host  | 设置绑定的ip地址，可以是ipv4或ipv6的，默认为0.0.0.0。 |
| network.publish_host  | 设置其它节点和该节点交互的ip地址，如果不设置它会自动判断，值必须是个真实的ip地址。 |
| network.host  | 这个参数是用来同时设置<span style='color:red;'>bind_host</span>和<span style='color:red;'>publish_host</span>上面两个参数。 |
| transport.tcp.port  | 设置节点间交互的tcp端口，默认是9300。 |
| transport.tcp.compress  | 设置是否压缩tcp传输时的数据，默认为false，不压缩。 |
| http.port  | 设置对外服务的http端口，默认为9200。 |
| http.max_content_length  | 设置内容的最大容量，默认<span style='color:red;'>100mb</span>。 |
| http.enabled  | 是否使用http协议对外提供服务，默认为true，开启。 |
| gateway.type  | gateway的类型，默认为local即为本地文件系统，可以设置为本地文件系统，分布式文件系统，hadoop的HDFS，和amazon的s3服务器，其它文件系统的设置。 |
| gateway.recover_after_nodes  | 设置集群中N个节点启动时进行数据恢复，默认为1。 |
| gateway.recover_after_time  | 设置初始化数据恢复进程的超时时间，默认是5分钟。 |
| gateway.expected_nodes  | 设置这个集群中节点的数量，默认为2，一旦这N个节点启动，就会立即进行数据恢复。 |
| cluster.routing.allocation.node_initial_primaries_recoveries  | 初始化数据恢复时，并发恢复线程的个数，默认为4。 |
| cluster.routing.allocation.node_concurrent_recoveries  | 添加删除节点或负载均衡时并发恢复线程的个数，默认为4。 |
| indices.recovery.max_size_per_sec  | 设置数据恢复时限制的带宽，如入100mb，默认为0，即无限制。 |
| indices.recovery.concurrent_streams  | 设置这个参数来限制从其它分片恢复数据时最大同时打开并发流的个数，默认为5。 |
| discovery.zen.minimum_master_nodes  | 设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。默认为1，对于大的集群来说，可以设置大一点的值（2-4） |
| discovery.zen.ping.timeout  | 设置集群中自动发现其它节点时ping连接超时时间，默认为3秒，对于比较差的网络环境可以高点的值来防止自动发现时出错。 |
| discovery.zen.ping.multicast.enabled  | 设置是否打开多播发现节点，默认是true。 |
| discovery.zen.ping.unicast.hosts  | 设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点。 |
| threadpool.search.type  | 查询线程池配置（fixed 固定） |
| threadpool.search.min  | 最小线程数 |
| threadpool.search.max  | 最大线程数 |
| threadpool.search.queue_size  | 线程池队列大小 |
| index.store.type  | ES索引存储 memory（内存） |
| index.mapper.dynamic  | 禁止自动创建mapping（默认false） |
| index.query.parse.allow_unmapped_fields  | 不能查找没有在mapping中定义的属性 |

