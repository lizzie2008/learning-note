# 简介

## OpenAPI 

Open API 即开放 API，也称开放平台。 所谓的开放 API（OpenAPI）是服务型网站常见的一种应用，网站的服务商将自己的网站服务封装成一系列
API（Application Programming Interface，应用编程接口）开放出去，供第三方开发者使用，这种行为就叫做开放网站的 API，所开放的 API 就被称作 OpenAPI（开放 API ）。

## RESTful API

Representational State Transfer，翻译是”表现层状态转化”。可以总结为一句话：REST 是所有 Web 应用都应该遵守的架构设计指导原则。
面向资源是 REST 最明显的特征，对于同一个资源的一组不同的操作。资源是服务器上一个可命名的抽象概念，资源是以名词为核心来组织的，首先关注的是名词。REST 要求，必须通过统一的接口来对资源执行各种操作。对于每个资源只能执行一组有限的操作。

什么是 RESTful API？
符合 REST 设计标准的 API，即 RESTful API。REST 架构设计，遵循的各项标准和准则，就是 HTTP 协议的表现，换句话说，HTTP 协议就是属于 REST 架构的设计模式。比如，无状态，请求-响应。。。

## 简单实践

那如何构建咱们自己的Open API，这里做了简单的代码示例，包括基础的权限验证、限流控制，方便笔者自己构建其他应用服务时的调用。

- 部署环境（阿里云ECS服务器）
  - 操作系统：Centos7.1
  - 容器管理：Docker version 1.13.1
  - 微服务注册中心镜像：webapp/eureka-server
  - 微服务配置中心镜像：webapp/config-server
  - 微服务API应用镜像：webapp/open-api  
  - 微服务API网关镜像：webapp/api-gateway
- 源码：[https://github.com/lizzie2008/spring-cloud-app.git](https://github.com/lizzie2008/spring-cloud-app.git)

# API工程

## 创建工程

- 创建Gateway工程（open-api），引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
</dependency>
```

## 暴露API

open-api工程很简单，实现业务逻辑，对外暴露接口即可，这里我们简单示例，新建一个测试Controller，返回一行文本。

```java
@RestController
@RequestMapping("/v1")
public class TestController {

    @GetMapping("/info")
    public String info(){
        return "Hello World!";
    }
}
```

启动服务，可以通过 http://localhost:8081/v1/info 正常访问。

# Gateway工程

## 创建工程

创建Gateway工程（api-gateway），导入相关依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

## 访问权限控制

- 数据库建权限表

```mysql
CREATE TABLE `access_info` (
  `access_key` varchar(32) NOT NULL COMMENT '访问码',
  `access_desc` varchar(32) NOT NULL COMMENT '访问说明',
  `visit_module` varchar(32) NOT NULL COMMENT '访问模块',
  `access_status` tinyint(3) NOT NULL DEFAULT '0' COMMENT '访问状态, 0：不允许访问 1：允许访问',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`access_key`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

- 从数据库获取权限

```java
@Data
@Entity
public class AccessInfo {

    /**
     * 访问码.
     */
    @Id
    private String accessKey;

    /**
     * 访问说明.
     */
    private String accessDesc;

    /**
     * 访问模块.
     */
    private String visitModule;

    /**
     * 访问状态, 0：不允许访问 1：允许访问
     */
    private AccessStatus accessStatus;


    /**
     * 创建时间.
     */
    private Date createTime;

    /**
     * 更新时间.
     */
    private Date updateTime;
}

@Repository
public interface AccessInfoRepository extends JpaRepository<AccessInfo, String> {

}

@Service
public class AccessInfoService {

    private final AccessInfoRepository accessInfoRepository;

    public AccessInfoService(AccessInfoRepository accessInfoRepository) {
        this.accessInfoRepository = accessInfoRepository;
    }

    /**
     * 获取所有访问权限信息
     *
     * @return
     */
    public List<AccessInfo> findAll() {
        return accessInfoRepository.findAll();
    }
}
```

- 新建`AccessFilter` ，继承`ZuulFilter`，来实现权限验证

```java
@Component
public class AccessFilter extends ZuulFilter {

    private final AccessInfoService accessInfoService;

    public AccessFilter(AccessInfoService accessInfoService) {
        this.accessInfoService = accessInfoService;
    }

    @Override
    public String filterType() {
        return PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext currentContext = RequestContext.getCurrentContext();
        HttpServletRequest request = currentContext.getRequest();

        if (!isAuthorized(request)) {
            HttpStatus httpStatus = HttpStatus.UNAUTHORIZED;
            currentContext.setSendZuulResponse(false);
            currentContext.setResponseStatusCode(httpStatus.value());
        }
        return null;
    }

    /**
     * 判断请求是否有权限
     *
     * @param request
     * @return
     */
    private boolean isAuthorized(HttpServletRequest request) {
        // 检查请求参数是否包含 access_key
        String access_key = request.getParameter("access_key");
        if (!StringUtils.isEmpty(access_key)) {
            // 检查 access_key 是否匹配
            List<AccessInfo> accessInfos = accessInfoService.findAll();
            Optional<AccessInfo> accessInfo = accessInfos.stream()
                    .filter(s -> access_key.equals(s.getAccessKey())).findAny();
            if (accessInfo.isPresent()) {
                return true;
            }
            return false;
        }
        return false;
    }
}
```

- 启动网关服务，访问 http://localhost:8080/open-api/v1/info
  - 如果请求参数不带`access_key`，网关服务会直接返回 401 未授权的错误；
  - 如果请求参数带`access_key`，但是与我们数据库的安全验证不匹配，网关服务也会直接返回 401 错误；

![image-20200113164209186](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200113164212-565594.png)  

- 请求`access_key`，也通过后台数据库验证，则调用成功

![image-20200113164052292](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200113164056-732474.png) 

## 权限缓存

以上已经实现了基本权限的验证，但是每次api的请求，都会进行数据库的校验。

```bash
2020-01-13 16:44:43.591  INFO 25028 --- [trap-executor-0] c.n.d.s.r.aws.ConfigClusterResolver      : Resolving eureka endpoints via configuration
Hibernate: select accessinfo0_.access_key as access_k1_0_, accessinfo0_.access_desc as access_d2_0_, accessinfo0_.access_status as access_s3_0_, accessinfo0_.create_time as create_t4_0_, accessinfo0_.update_time as update_t5_0_, accessinfo0_.visit_module as visit_mo6_0_ from access_info accessinfo0_
Hibernate: select accessinfo0_.access_key as access_k1_0_, accessinfo0_.access_desc as access_d2_0_, accessinfo0_.access_status as access_s3_0_, accessinfo0_.create_time as create_t4_0_, accessinfo0_.update_time as update_t5_0_, accessinfo0_.visit_module as visit_mo6_0_ from access_info accessinfo0_
Hibernate: select accessinfo0_.access_key as access_k1_0_, accessinfo0_.access_desc as access_d2_0_, accessinfo0_.access_status as access_s3_0_, accessinfo0_.create_time as create_t4_0_, accessinfo0_.update_time as update_t5_0_, accessinfo0_.visit_module as visit_mo6_0_ from access_info accessinfo0_

```

实际生产中肯定不能这么操作，对数据库的压力太大，所以，我们要对权限验证的数据进行缓存。

- 首先，在启动类上增加`@EnableCaching`注解

```java
@EnableDiscoveryClient
@EnableZuulProxy
@SpringBootApplication
@EnableCaching
public class ApiGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }

}
```

- `AccessInfo`需要实现`Serializable`接口，方便序列化后保存在redis中。

```java
@Data
@Entity
public class AccessInfo implements Serializable {
    //...
}
```

- 获取所有访问权限信息的方法上增加缓存处理

```java
@Cacheable(value = "api-gateway:accessInfo")
public List<AccessInfo> findAll() {
    return accessInfoRepository.findAll();
}
```

- 重新启动服务后，多调用几次api接口，发现第一次加载时会调用一次数据库，后面都是取缓存中的权限信息，不再查询数据库。

## 限流控制

RateLimiter是guava提供的基于令牌桶算法的实现类，可以非常简单的完成限流特技，并且根据系统的实际情况来调整生成token的速率。

- 新建`RateLimitFilter`继承`ZuulFilter`，定义一个RateLimiter，这里为了测试方便，每秒设置最多2个请求

```java
/**
 * 限流
 */
@Component
public class RateLimitFilter extends ZuulFilter {

    //每秒产生N个令牌
    private static final RateLimiter rateLimiter = RateLimiter.create(2);

    @Override
    public String filterType() {
        return PRE_TYPE;
    }


    @Override
    public int filterOrder() {
        return SERVLET_DETECTION_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }


    @Override
    public Object run() {
        if (!rateLimiter.tryAcquire()) {
            RequestContext currentContext = RequestContext.getCurrentContext();
            HttpStatus httpStatus = HttpStatus.TOO_MANY_REQUESTS;
            currentContext.setSendZuulResponse(false);
            currentContext.setResponseStatusCode(httpStatus.value());
        }

        return null;
    }
}
```

- 重启服务后，1s内如果多次访问接口，会提示 429 Too Many Requests错误，这样限流的功能就完成了

![image-20200113184107059](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200113184107-778895.png) 

# 部署环境

## 微服务注册

将打包的jar文件生成docker镜像，然后部署在个人服务器上，之前笔者已经部署过服务注册中心（eureka-server）和统一配置中心（config-server），所以把两个新应用注册并部署即可。

这里是微服务部署，将服务注册到服务中心，并从统一配置中心获取配置属性，后面可以通过实例名称来进行访问。

- 配置open-api工程

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://eureka1:8761/eureka/,http://eureka2:8762/eureka/ # 指定服务注册地址

spring:
  application:
    name: open-api  # 应用名称

server:
  port: 8081
```

- 配置api-gateway工程

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://eureka-server:8761/eureka/ #指定服务注册地址

spring:
  application:
    name: api-gateway  #应用名称
  cloud:
    config:
      discovery:
        enabled: true
        service-id: config-server
```

![image-20200116154951810](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200116154955-275522.png)  

## 启动服务

依次启动eureka-server、config-server、open-api、api-gateway服务，这样我们就可以通过访问域名地址来访问自己的API了。这里尤其注意open-api启动后再启动api-gateway服务，不然api-gateway服务在eureka-server上无法找到open-api服务，所以不会配置默认的路由规则，会导致服务不可用。

![image-20200116165920076](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200116165920-459496.png) 