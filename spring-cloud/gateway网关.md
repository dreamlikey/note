#### 简单示例

创建spring-cloud-gateway项目



##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.wdq.yun</groupId>
        <artifactId>yun</artifactId>
        <version>1.0</version>
    </parent>

    <groupId>com.wdq.yun</groupId>
    <artifactId>yun-gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>yun-gateway</name>
    <description>网关</description>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-contract-stub-runner</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-web</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```



clooud包注意版本冲突



##### spring boot yml文件

```yml
spring:
  application:
    name: gateway
  main:
    allow-bean-definition-overriding: true
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #启用网关
#     简单路由配置      
      routes:
      - id: shop_route
        uri: http://192.168.8.112:8083
#        uri: http://spring.io/
        predicates:
        - Path=/shopapi/**
      - id: goods_route
        uri: http://192.168.8.112:8084
        predicates:
        - Path=/goods/**

server:
  port: 8086
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8080/eureka
logging:
  level:
    org.springframework.cloud.gateway: debug
```



访问：<http://localhost:8086/shopapi/redis/get>

网关路由到： http://192.168.8.112:8083/shopapi/redis/get

#### 路由

##### Host路由

**application.yml**

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org,**.anotherhost.org
```



##### 请求方式路由

​	该路由配置需要一个http方法的参数

**application.yml**

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET
```

该路由当请求方法为GET方法时匹配

##### 路径路由

**application.yml**

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Path=/foo/{segment},/bar/{segment}
```

请求路径为`/foo/1` or `/foo/bar` or /bar/baz 时匹配路由规则

##### 查询路由



##### 权重路由

**application.yml**

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: weight_high
        uri: https://weighthigh.org
        predicates:
        - Weight=group1, 8
      - id: weight_low
        uri: https://weightlow.org
        predicates:
        - Weight=group1, 2
```

80%路由到[https://weighthigh.org](https://weighthigh.org/) 

20%路由到https://weightlow.org



#### 网关过滤器

#### 全局过滤器

#### 动态路由

