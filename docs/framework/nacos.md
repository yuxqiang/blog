# nacos源码分析
## nacos服务注册流程
### Nacos Server 的注册表结构 如下

``` java
    //com.alibaba.nacos.naming.core.ServiceManager
     /**
     * Map(namespace, Map(group::serviceName, Service)).
     */
    private final Map<String, Map<String, Service>> serviceMap = new ConcurrentHashMap<>();
```
这是一个典型的Map结构 key是namespace 值又是一个Map map的key是group和serverName名称组成 值是service实体
service又是一个map对象
``` java
    //package com.alibaba.nacos.naming.core.Service
     /**
     * Map(namespace, Map(group::serviceName, Service)).
     */
 
    private Map<String, Cluster> clusterMap = new HashMap<>();
```
Map的Key值为Cluster的名字，Value为Cluster对象，Cluster对象中有两个Set的数据结构，用来存储Instance，这个Instance才是真正的客户端注册过来的实例信息。
``` java
//com.alibaba.nacos.naming.core.Cluster
    //持久化实例
    private Set<Instance> persistentInstances = new HashSet<>();
    //临时实例
    private Set<Instance> ephemeralInstances = new HashSet<>();
```
这里有两个Set，一个是用来存储临时实例，一个是用来存储持久化实例，有个关键点，什么情况会存储在临时实例，什么情况下会存储持久化实例，这个是由客户端的配置来决定的，默认情况下客户端配置ephemeral=true，如果你想把实例用持久化的方式来存储，可以设置ephemeral=false，这样在客户端发起注册的时候会把这个参数带到Nacos Server，Nacos Server就会按照持久化的方式来存储。
注意：Nacos目前持久化存储的方式采用的是本地文件存储的方式。

### 客户端注册流程如下



