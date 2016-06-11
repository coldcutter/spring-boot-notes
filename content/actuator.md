# 7. Actuator

启用Actuator，你只要在build.gradle中加入：

```
compile 'org.springframework.boot:spring-boot-starter-actuator'
```

Spring Boot Actuator提供了一系列RESTful接口：

| HTTP方法 | 路径 | 描述 |
| -- | -- | -- |
| GET | /autoconfig | 自动配置报告，哪些自动配置生效或无效 |
| GET | /configprops | 各种配置属性 |
| GET | /beans | Spring应用上下文中所有bean以及它们之间的依赖关系。 |
| GET | /dump | Retrieves a snapshot dump of thread activity. |
| GET | /env | 各种环境属性 |
| GET | /env/{name} | 根据名字检索环境属性 |
| GET | /health | Reports health metrics for the application, as provided by HealthIndicator implementations. |
| GET | /info | Retrieves custom information about the application, as provided by any properties prefixed with info. |
| GET | /mappings | Describes all URI paths and how they’re mapped to controllers (including Actuator endpoints). |
| GET | /metrics | Reports various application metrics such as memory usage and HTTP request counters. |
| GET | /metrics/{name} | Reports an individual application metric by name. |
| POST | /shutdown | Shuts down the application; requires that endpoints.shutdown.enabled be set to true. |
| GET | /trace | Provides basic trace information (timestamp, headers, and so on) for HTTP requests. |

这些信息主要分为三类：配置（configuration）、指标（metrics）、其他（miscellaneous）。为了安全，任何名字或者名字最后部分是“password”、“secret”或者“key”的属性，都会被打星显示.

## 7.1 配置信息

Spring应用的一大问题就是你不太清楚有哪些bean以及它们的注入关系，特别是当你使用了Spring Boot自动配置以后，不过有了Actuator，你可以用/beans获得一份bean清单：