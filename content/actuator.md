# 7. Actuator

启用Actuator，你只要在build.gradle中加入：

```
compile 'org.springframework.boot:spring-boot-starter-actuator'
```

Spring Boot Actuator提供了一系列RESTful接口：

| HTTP方法 | 路径 | 描述 |
| -- | -- | -- |
| GET | /beans | Spring应用上下文中所有bean以及它们之间的依赖关系 |
| GET | /autoconfig | 自动配置报告，哪些自动配置生效或无效 |
| GET | /env | 各种环境属性 |
| GET | /env/{name} | 根据名字检索环境属性 |
| GET | /configprops | 各种配置属性 |
| GET | /mappings | URI路径到控制器的映射 |
| GET | /metrics | 各种指标 |
| GET | /metrics/{name} | 根据名字检索指标 |
| GET | /dump | Retrieves a snapshot dump of thread activity. |
| GET | /health | Reports health metrics for the application, as provided by HealthIndicator implementations. |
| GET | /info | Retrieves custom information about the application, as provided by any properties prefixed with info. |
| POST | /shutdown | Shuts down the application; requires that endpoints.shutdown.enabled be set to true. |
| GET | /trace | Provides basic trace information (timestamp, headers, and so on) for HTTP requests. |

这些信息主要分为三类：配置（configuration）、指标（metrics）、其他（miscellaneous）。为了安全，任何名字或者名字最后部分是“password”、“secret”或者“key”的属性，都会被打星显示。

## 7.1 配置信息

包括/beans、/autoconfig、/env、/env/{name}、/configprops、/mappings

## 7.2 指标信息

/metrics接口返回各种指标，按照前缀分类：

| 分类 | 前缀 | 描述 |
| -- | -- | -- |
| 垃圾收集器 | gc.* | mark-sweep和scavenge垃圾收集器的垃圾收集计数和持续时间 (from java.lang.management.GarbageCollectorMXBean) |
| 内存 | mem.* | 分配给应用的内存和空闲的内存 (from java.lang.Runtime) |
| 堆 | heap.* | 当前内存使用 (from java.lang .management.MemoryUsage) |
| 类加载器 | classes.* | JVM类加载器加载和未加载的类数量  (from java.lang.management.ClassLoadingMXBean) |
| 系统 | processors，uptime，instance.uptime，systemload.average | 处理器数量 (from java.lang.Runtime)，运行时间 (from java.lang.management.RuntimeMXBean)，平均系统负载 (from java.lang.management.OperatingSystemMXBean) |
| 线程池 | threads.* | 线程、守护线程的数量，以及JVM启动以来的线程峰值 (from java.lang.management.ThreadMXBean) |
| 数据源 | datasource.* | 数据源连接数量 (from the data source’s metadata，仅当存在datasource bean时才有) |
| HTTP会话 | httpsessions.* | 活跃的和最大的会话数量 (from the embedded Tomcat bean，仅当有内置服务器时才有) |
| HTTP | counter.status.*，gauge.response.* | HTTP请求的各种度量和统计 |