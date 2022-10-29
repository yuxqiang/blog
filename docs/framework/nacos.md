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
1.基于client-server注册模式 整合springcloud集成nacos注册中心是使用这种注册流程的

nacos-client--------》nacos-server 通过http请求代码如下
``` java
//com.alibaba.nacos.client.naming.NacosNamingService 核心方法注册实例信息
   public void registerInstance(String serviceName, String groupName, Instance instance) throws NacosException {
        String groupedServiceName = NamingUtils.getGroupedName(serviceName, groupName);
        if (instance.isEphemeral()) {
           //判断是否临时实例 临时实例添加心跳检测线程任务
            BeatInfo beatInfo = this.beatReactor.buildBeatInfo(groupedServiceName, instance);
            this.beatReactor.addBeatInfo(groupedServiceName, beatInfo);
        }

        this.serverProxy.registerService(groupedServiceName, groupName, instance);
    }
```
接下来查看serverProxy这个类 具体如下所示
``` java
//主要功能是拼接请求参数去请求 nacos-server 的接口
public void registerService(String serviceName, String groupName, Instance instance) throws NacosException {
        LogUtils.NAMING_LOGGER.info("[REGISTER-SERVICE] {} registering service {} with instance: {}", new Object[]{this.namespaceId, serviceName, instance});
        Map<String, String> params = new HashMap(16);
        params.put("namespaceId", this.namespaceId);
        params.put("serviceName", serviceName);
        params.put("groupName", groupName);
        params.put("clusterName", instance.getClusterName());
        params.put("ip", instance.getIp());
        params.put("port", String.valueOf(instance.getPort()));
        params.put("weight", String.valueOf(instance.getWeight()));
        params.put("enable", String.valueOf(instance.isEnabled()));
        params.put("healthy", String.valueOf(instance.isHealthy()));
        params.put("ephemeral", String.valueOf(instance.isEphemeral()));
        params.put("metadata", JacksonUtils.toJson(instance.getMetadata()));
        this.reqApi(UtilAndComs.nacosUrlInstance, params, "POST");
    }
```
接下来我们看reqApi这个方法
```java
// 这是客户端请求nacos-server核心代码 
public String reqApi(String api, Map<String, String> params, Map<String, String> body, List<String> servers,String method) throws NacosException {

        params.put(CommonParams.NAMESPACE_ID, getNamespaceId());
        if (CollectionUtils.isEmpty(servers) && StringUtils.isBlank(nacosDomain)) {
        throw new NacosException(NacosException.INVALID_PARAM, "no server available");
        }

        NacosException exception = new NacosException();

        //如果是单个注册中心走这里
        if (StringUtils.isNotBlank(nacosDomain)) {
        for (int i = 0; i < maxRetry; i++) {//默认重试三次
        try {
        return callServer(api, params, body, nacosDomain, method);
        } catch (NacosException e) {
        exception = e;
        if (NAMING_LOGGER.isDebugEnabled()) {
        NAMING_LOGGER.debug("request {} failed.", nacosDomain, e);
        }
        }
        }
        } else {  //如果是多个注册中心走这里 配置了多个注册中心
        Random random = new Random(System.currentTimeMillis());
        int index = random.nextInt(servers.size());

        for (int i = 0; i < servers.size(); i++) {
        String server = servers.get(index);
        try {
        return callServer(api, params, body, server, method);
        } catch (NacosException e) {
        exception = e;
        if (NAMING_LOGGER.isDebugEnabled()) {
        NAMING_LOGGER.debug("request {} failed.", server, e);
        }
        }
        index = (index + 1) % servers.size();
        }
        }

        NAMING_LOGGER.error("request: {} failed, servers: {}, code: {}, msg: {}", api, servers, exception.getErrCode(),
        exception.getErrMsg());

        throw new NacosException(exception.getErrCode(),
        "failed to req API:" + api + " after all servers(" + servers + ") tried: " + exception.getMessage());

        }
```
以上就是clien的注册流程结束了 接下来我们查看一下nacos-server端代码








