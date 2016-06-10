# 7. Actuator

启用Actuator，你只要在build.gradle中加入：

```
compile 'org.springframework.boot:spring-boot-starter-actuator'
```

Spring Boot Actuator提供了一系列RESTful接口：

| HTTP方法 | 路径 | 描述 |
| -- | -- | -- |
| GET | /autoconfig | Provides an auto-configuration report describing what auto- configuration conditions passed and failed. |
| GET | /configprops | Describes how beans have been injected with configuration properties (including default values). |
| GET | /beans | Describes all beans in the application context and their relationship to each other. |
| GET | /dump | Retrieves a snapshot dump of thread activity. |
| GET | /env | Retrieves all environment properties. |
| GET | /env/{name} | Retrieves a specific environment value by name. |
| GET | /health | Reports health metrics for the application, as provided by HealthIndicator implementations. |
| GET | /info | Retrieves custom information about the application, as provided by any properties prefixed with info. |
| GET | /mappings | Describes all URI paths and how they’re mapped to controllers (including Actuator endpoints). |
| GET | /metrics | Reports various application metrics such as memory usage and HTTP request counters. |
| GET | /metrics/{name} | Reports an individual application metric by name. |
| POST | /shutdown | Shuts down the application; requires that endpoints.shutdown.enabled be set to true. |
| GET | /trace | Provides basic trace information (timestamp, headers, and so on) for HTTP requests. |

这些信息主要分为三类：配置（configuration）、指标（metrics）、其他（miscellaneous）。

## 7.1 配置信息

Spring应用的一大问题就是你不太清楚有哪些bean以及它们的注入关系，特别是当你使用了Spring Boot自动配置以后，不过有了Actuator，你可以用/beans获得一份bean清单：