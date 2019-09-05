## 概述
- 为什么需要本地缓存？

在系统中，访问十分频繁的热点数据（例如数据字典数据、国家标准行政区域数据），往往需要将这些数据放入分布式缓存中，但为了减少网络传输，加快响应速度，缓解分布式缓存读压力，会把这些数据缓存到本地JVM中，大多是先取本地缓存中，再取分布式缓存中的数据，这样就会形成多级缓存。

- 缓存相关文章：
    - [你应该知道缓存进化史](https://juejin.im/post/5b7593496fb9a009b62904fa)
    - [如何优雅的设计和使用缓存？](https://juejin.im/post/5b849878e51d4538c77a974a)

- 为什么是Caffeine

[Caffeine](https://github.com/ben-manes/caffeine)是一款基于Java 8的高性能缓存库，提供了近似最佳的命中率；Spring 4.3 & Boot 1.4中用Caffeine取代了Guava。

## 引入依赖
实际上caffeine等均已经在`org.springframework.boot:spring-boot-dependencies:pom:2.0.6.RELEASE`中做了依赖管理，此处不需要指定版本，其是`org.springframework.boot:spring-boot-starter-parent:pom:2.0.6.RELEASE`的父级POM，也是所有Spring Boot项目的顶级POM。
> `spring-boot-starter-actuator`是为后续对Cache监控而引入的

```xml
<!-- Caffeine -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

## 方案一：Yml配置
- 如果Caffeine存在，则`CaffeineCacheManager`(由`spring-boot-starter-cache`"Starter"提供)将会自动配置。
- 通过设置`spring.cache.cache-names`将会在服务启动时自动创建缓存
- 同时可以通过如下几种方式客户化缓存特性，优先级如下
    - 属性`spring.cache.caffeine.spec`
    - 存在Bean `com.github.benmanes.caffeine.cache.CaffeineSpec`
    - 存在Bean `com.github.benmanes.caffeine.cache.Caffeine`
- 示例：如下配置创建了两个缓存`cache1`及`cache2`，最大缓存大小为500，TTL(Time to live)10分钟
```yaml
spring.cache.cache-names=cache1,cache2
spring.cache.caffeine.spec=maximumSize=500,expireAfterAccess=600s
```

## 方案二：代码实现
- 增加配置类
    - 首先通过`@EnableCaching`开启缓存
    - 通过`enum`手动定义缓存配置，具体Caffeine的缓存配置项可以参考`com.github.benmanes.caffeine.cache.CaffeineSpec`或者`com.github.benmanes.caffeine.cache.Caffeine`
    - 通过客户化`caffeineCacheManager`手动将`enum`中的配置创建为Caffeine缓存并纳入管理
```java
/**
 * 配置类
 *
 * @author allen.liu
 * @date 2019/6/3
 */
@Configuration
@EnableCaching
public class CaffeineCacheConfiguration {
    /**
     * 缓存配置
     */

    public static final int DEFAULT_MAXIMUMSIZE = 10000;
    public static final int DEFAULT_TTL = 600;
    /**
     * Caffeine Cache 配置容器
     */
    public enum CaffeineCacheConfigContainer {
        /**
         * accessToken缓存, TTL为1天，最大容量为50000
         */
        accessToken(86400, 50000),
        ;
        // 最大數量
        private int maximumSize = DEFAULT_MAXIMUMSIZE;
        // 过期时间（秒），Time to live
        private int ttl = DEFAULT_TTL;

        CaffeineCacheConfigContainer() {
        }
        CaffeineCacheConfigContainer(int ttl) {
            this.ttl = ttl;
        }
        CaffeineCacheConfigContainer(int ttl, int maximumSize) {
            this.ttl = ttl;
            this.maximumSize = maximumSize;
        }

        public int getMaximumSize() {
            return maximumSize;
        }
        public int getTtl() {
            return ttl;
        }
    }

    /**
     * 创建基于Caffeine的Cache Manager
     * @return Caffeine Cache Manager
     */
    @Bean
    @Primary
    public CacheManager caffeineCacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        List<CaffeineCache> caches = new ArrayList<>();
        for(CaffeineCacheConfigContainer caffeineCacheConfig : CaffeineCacheConfigContainer.values()){
            caches.add(new CaffeineCache(caffeineCacheConfig.name(),
                    Caffeine.newBuilder().recordStats()
                            .expireAfterWrite(caffeineCacheConfig.getTtl(), TimeUnit.SECONDS)
                            .maximumSize(caffeineCacheConfig.getMaximumSize())
                            .build())
            );
        }
        cacheManager.setCaches(caches);
        return cacheManager;
    }
}
```

## 缓存使用
- `@Cacheable`中`value`与如上配置中缓存的名称保持一致；`key`为缓存的Key值，可通过SPEL表达式获取方法及参数的相关信息；
- `@CacheEvict`定义了缓存的淘汰策略，例如，此处传入了参数`refresh`
```java
    /**
     * 增加本地缓存
     *
     * @since 2019/9/2
     * @author allen.liu
     *
     * @param httpAuthentication 认证信息
     * @return Access Token
     */
    @Cacheable(value = "accessToken", key="#httpAuthentication.accessTokenCacheKey")
    @CacheEvict(value = "accessToken", key="#httpAuthentication.accessTokenCacheKey", condition = "#refresh")
    @Override
    public String getToken(HttpAuthentication httpAuthentication, boolean refresh) {
        LOGGER.info("enter get token with parameter [{}].", httpAuthentication);
        Assert.hasText(httpAuthentication.getAccessTokenUrl(), Constant.ErrorCode.DATA_INVALID);

        // 获取相关参数
        Map<String, String> accessTokenRequestParamMap = new HashMap<>(16);
        this.buildGrantType(httpAuthentication, accessTokenRequestParamMap);

        // 设置权限范围
        String scope = httpAuthentication.getScope();
        if (StringUtils.isNotBlank(scope)) {
            accessTokenRequestParamMap.put("scope", scope);
        }

        // 获取accessToken
        return this.buildAccessToken(httpAuthentication.getAccessTokenUrl(), accessTokenRequestParamMap);
    }
```

## 缓存验证
可通过两次请求的LOGGER日志观察，第一次将会打印方法的第一条日志，第二次启用缓存之后，将不会再打印日志。

## 缓存监控
只要开启metrics管理端点，则缓存的自动化配置将会自动纳入到Metrics的管理(默认包含了Caffeine缓存，具体可参考自动化配置[org.springframework.boot.actuate.autoconfigure.metrics.cache.CacheMetricsAutoConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-actuator-autoconfigure/src/main/java/org/springframework/boot/actuate/autoconfigure/metrics/cache/CacheMetricsAutoConfiguration.java))，因此，只需要开启Metrics的运维端点即可使用。
> Metrics指标参考：[Spring Boot默认指标从哪来？](https://www.cnblogs.com/liululee/p/11410493.html)

```yaml
management:
  server:
    port: 8151
  endpoints:
    web:
      exposure:
        include: info,health,monitoring,metrics,caches
```

> 实践看来：Spring Boot 2.0.6.RELEASE版本配置`include: '*'`似乎无法生效，因此，单独配置各个端点。

http://localhost:8151/actuator/metrics
```json
{
    "names": [
        "http.server.requests",
        "cache.puts",
        "cache.size",
        "cache.evictions",
        "cache.eviction.weight",
        "cache.gets"
    ]
}
```

http://localhost:8151/actuator/metrics/cache.gets
> `measurements.value`将会随缓存命中和获取而增加

```json
{
    "name": "cache.gets",
    "description": "The number of times cache lookup methods have returned a cached value.",
    "baseUnit": null,
    "measurements": [
        {
            "statistic": "COUNT",
            "value": 3
        }
    ],
    "availableTags": [
        {
            "tag": "result",
            "values": [
                "hit",
                "miss"
            ]
        },
        {
            "tag": "cache",
            "values": [
                "accessToken"
            ]
        },
        {
            "tag": "name",
            "values": [
                "accessToken"
            ]
        },
        {
            "tag": "cacheManager",
            "values": [
                "caffeine"
            ]
        }
    ]
}
```

## 如何关闭缓存监控端点
- 目前无法关闭特定Metrics指标，只能启用/关闭特定端点，例如，如下Spring Boot配置
```yaml
management.endpoints.autoconfig.enabled=false
management.endpoints.beans.enabled=false
management.endpoints.configprops.enabled=false
management.endpoints.dump.enabled=false
management.endpoints.env.enabled=false
management.endpoints.health.enabled=true
management.endpoints.info.enabled=true
management.endpoints.metrics.enabled=false
management.endpoints.mappings.enabled=false
management.endpoints.shutdown.enabled=false
management.endpoints.trace.enabled=false
```

- 但可以通过将相关自动配置类加入到启动排除列表，例如，通过如下方式关闭缓存的Metrics指标

```java
import org.springframework.boot.actuate.autoconfigure.metrics.cache.CacheMetricsAutoConfiguration;

@EnableAutoConfiguration(exclude = {CacheMetricsAutoConfiguration.class})
```