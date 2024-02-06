# 网关的基本功能。
统一入口。网关是所有微服务的门户，路由转发仅仅是最基本的功能，除此之外还有其他的一些功能，比如：认证、鉴权、熔断、限流、日志监控等等.........

# springgateway基本组成
路由（route）：gateway的基本构建模块。它由ID、目标URI、断言集合和过滤器集合组成。如果聚合断言结果为真，则匹配到该路由。
断言（Predicate ）：参照Java8的新特性Predicate，允许开发人员匹配HTTP请求中的任何内容，比如头或参数。
过滤器（filter）：可以在返回请求之前或之后修改请求和响应的内容。

## 什么是Predict（断言）？
Predicate来自于java8的接口。Predicate接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将Predicate组合成其他复杂的逻辑（比如：与，或，非）。
可以用于接口请求参数校验、判断新老数据是否有变化需要进行更新操作。
Spring Cloud Gateway内置了许多Predict，这些Predict的源码在org.springframework.cloud.gateway.handler.predicate包中，有兴趣可以阅读一下。内置的一些断言如下图：

## 如何配置Predict
``` yaml
spring:
  application:
    name: gateway
  cloud:
    gateway:
      routes: #路由
        - id: tick-route
          filters: #过滤器
            - StripPrefix=1
          predicates: #断言
            - Path=/tick/**
          uri: lb://demo-tick1
        - id: tick-route2
          filters:
            - StripPrefix=1
          predicates:
            - name: Path
            - Path=/dick/**
          uri: lb://demo-dick
 ```
# 什么是过滤器？
Gateway的生命周期：
PRE：这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择 请求的微服务、记录调试信息等。
POST：这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。

Gateway 的Filter从作用范围可分为两种:

GatewayFilter：应用到单个路由或者一个分组的路由上（需要在配置文件中配置）。
GlobalFilter：应用到所有的路由上（无需配置，全局生效）

## 自定义局部过滤器
``` java
package com.maggie.gateway.config;

import jdk.nashorn.internal.runtime.logging.Logger;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

import java.util.Arrays;
import java.util.List;

@Component
@Slf4j
public class CustomerGatewayFilterFactory extends AbstractGatewayFilterFactory<CustomerGatewayFilterFactory.Config> {
    private static final String AUTHORIZE_TOKEN = "token";
    //构造函数，加载Config
    public CustomerGatewayFilterFactory() {
        //固定写法
        super(CustomerGatewayFilterFactory.Config.class);
        log.info("Loaded GatewayFilterFactory [Authorize]");
    }

    //读取配置文件中的参数 赋值到 配置类中
    @Override
    public List<String> shortcutFieldOrder() {
        //Config.enabled
        return Arrays.asList("enabled");
    }
    @Override
    public GatewayFilter apply(CustomerGatewayFilterFactory.Config config) {
        log.info("进入局部过滤器");
        return (exchange, chain) -> {
            //判断是否开启授权验证
            if (!config.isEnabled()) {
                return chain.filter(exchange);
            }

            ServerHttpRequest request = exchange.getRequest();
            HttpHeaders headers = request.getHeaders();
            //从请求头中获取token
            String token = headers.getFirst(AUTHORIZE_TOKEN);
            if (token == null) {
                //从请求头参数中获取token
                token = request.getQueryParams().getFirst(AUTHORIZE_TOKEN);
            }

            ServerHttpResponse response = exchange.getResponse();
            //如果token为空，直接返回401，未授权
            if (StringUtils.isEmpty(token)) {
                response.setStatusCode(HttpStatus.UNAUTHORIZED);
                //处理完成，直接拦截，不再进行下去
                return response.setComplete();
            }
            //授权正常，继续下一个过滤器链的调用
            return chain.filter(exchange);
        };
    }
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class Config {
        // 控制是否开启认证
        private boolean enabled;
    }
}
```
```yaml
spring:
  application:
    name: gateway
  cloud:
    gateway:
      routes:
        - id: tick-route
          filters:
            - Customer=true
            #- StripPrefix=1
          predicates:
            - Path=/tick/**
          uri: lb://demo-tick1
        - id: tick-route2
          filters:
             - Customer=true #名称保持一直和类名称
            #- StripPrefix=1
          predicates:
            - name: Path
            - Path=/dick/**
          uri: lb://demo-dick
    nacos:
      config:
        server-addr: 115.159.75.82:8848
        file-extension: yml
        group: gateway
        prefix: application
        username: nacos
        password: nacos
        access-key:
      discovery:
        #ephemeral: false
        server-addr: 115.159.75.82:8848
server:
  port: 8081
```
局部过滤器到此结束。

# 全局过滤器
全局过滤器只需要实现 GlobalFilter 就行 不需要像局部过滤器有严格的规定