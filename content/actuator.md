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
| GET | /trace | 最近100条HTTP请求追踪（数据在内存里） |
| GET | /dump | 线程活动的快照 |
| GET | /health | 应用健康指标，由HealthIndicator接口的实现提供 |
| POST | /shutdown | 关闭应用 |
| GET | /info | 提供关于应用的自定义信息 |

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

/health接口返回各种健康指标：

| Health indicator | Key | Reports |
| -- | -- | -- |
| ApplicationHealthIndicator | none | Always “UP” |
| DataSourceHealthIndicator | db | “UP” and database type if the database can be contacted; “DOWN” status otherwise |
| DiskSpaceHealthIndicator | diskSpace | “UP” and available disk space, and “UP” if available space is above a threshold; “DOWN” if there isn’t enough disk space |
| JmsHealthIndicator | jms | “UP” and JMS provider name if the message broker can be contacted; “DOWN” otherwise |
| MailHealthIndicator | mail | “UP” and the mail server host and port if the mail server can be contacted; “DOWN” otherwise |
| MongoHealthIndicator | mongo | “UP” and the MongoDB server version; “DOWN” otherwise |
| RabbitHealthIndicator | rabbit | “UP” and the RabbitMQ broker version; “DOWN” otherwise |
| RedisHealthIndicator | redis | “UP” and the Redis server version; “DOWN” otherwise |
| SolrHealthIndicator | solr | “UP” if the Solr server can be contacted; “DOWN” otherwise |

假设你需要关闭你的应用，比如在微服务架构中，你需要优雅地关闭其中一个应用实例，你可以这样：

```
curl -X POST http://localhost:8080/shutdown
```

显然，关闭应用是危险的行为，因此/shutdown默认是不启用的，你需要设置endpoints.shutdown.enabled为true才能开启，同时你应该注意/shutdown接口的权限。

/info接口返回应用的信息，默认返回空的JSON对象（{}），你通过配置info打头的属性来添加更多返回：

```
info:
  contact:
    email: support@myreadinglist.com
    phone: 1-888-555-1971
```

## 7.3 Remote Shell

Spring Boot集成了CRaSH（可内置在Java应用中的shell），并提供了一系列命令，首先加入依赖：

```
compile("org.springframework.boot:spring-boot-starter-remote-shell")
```

然后待应用起来之后，你可以使用SSH连上去（端口2000），密码就是log里打印出来的那串随机密码：

```
ssh user@localhost -p 2000
```

可以使用由Spring Boot提供的命令：autoconfig、beans、metrics（可以看到动态更新，Ctrl-C退出），但是并不是每个Web接口都有对应的命令，这时候你需要endpoint命令：

```
endpoint list
```

这会返回endpoint列表，不过不是url，而是它们的bean name：

```
requestMappingEndpoint
environmentEndpoint
healthEndpoint
beansEndpoint
infoEndpoint
metricsEndpoint
traceEndpoint
dumpEndpoint
autoConfigurationReportEndpoint
configurationPropertiesReportEndpoint
```

然后使用endpoint invoke命令（去掉“Endpoint”后缀）：

```
endpoint invoke health
```

## 7.4 自定义

### 7.4.1 修改endpoint IDs

每个Actuator endpoint都有一个ID来决定路径，比如/beans的默认ID就是beans，通过修改endpoints.endpoint-id.id属性就能修改路径，比如修改/shutdown为/kill：

```
endpoints:
  shutdown:
    id: kill
```

### 7.4.2 禁用（启用）endpoints

你还可以禁用某些endpoint：

```
endpoints:
  metrics:
    enabled: false
```

你可以禁用所有，并启用某些：

```
endpoints:
  enabled: false
  metrics:
    enabled: true
```

### 7.4.3 添加自定义metrics和gauges

比如我们需要记录用户保存了几次书到reading list，以及最后一次保存的时间，并通过/metrics暴露出来，Actuator为我们自动配置了一个CounterService实例，帮助完成计数功能：

```
package org.springframework.boot.actuate.metrics;

public interface CounterService {
  void increment(String metricName);
  void decrement(String metricName);
  void reset(String metricName);
}
```

还有一个GaugeService实例，帮助给一个指标设置一个值：

```
package org.springframework.boot.actuate.metrics;

public interface GaugeService {
  void submit(String metricName, double value);
}
```

你只需要注入这两个bean，然后直接调用方法即可：

```
@RequestMapping(method = RequestMethod.POST)
public String addToReadingList(Reader reader, Book book) {
  
  book.setReader(reader);
  readingListRepository.save(book);
  
  counterService.increment("books.saved");
  gaugeService.submit("books.last.saved", System.currentTimeMillis());
  
  return "redirect:/";
}
```

对于稍微复杂点的metrics，上面那两个Service估计不够用了，这时候你可以实现PublicMetrics接口：

```
package org.springframework.boot.actuate.endpoint;

public interface PublicMetrics {
  Collection<Metric<?>> metrics();
}
```

比如，我们要知道application context启动的时间，bean的数量，bean定义的数量以及注解为@Controller的bean的数量：

```
package readinglist;
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.endpoint.PublicMetrics;
import org.springframework.boot.actuate.metrics.Metric;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Controller;

@Component
public class ApplicationContextMetrics implements PublicMetrics {
  
  @Autowired
  private ApplicationContext context;
  
  @Override
  public Collection<Metric<?>> metrics() {
    List<Metric<?>> metrics = new ArrayList<Metric<?>>();
    metrics.add(new Metric<Long>("spring.context.startup-date", context.getStartupDate()));
    metrics.add(new Metric<Integer>("spring.beans.definitions", context.getBeanDefinitionCount()));
    metrics.add(new Metric<Integer>("spring.beans", context.getBeanNamesForType(Object.class).length));
    metrics.add(new Metric<Integer>("spring.controllers", context.getBeanNamesForAnnotation(Controller.class).length));
    return metrics;
  }
}
```

然后访问/metrics，就会显示我们自定义的指标：

```
{
  ...
  spring.context.startup-date: 1429398980443,
  spring.beans.definitions: 261,
  spring.beans: 272,
  spring.controllers: 2,
  books.count: 1,
  gauge.books.save.time: 1429399793260,
  ...
}
```

spring.controllers为2，是因为统计了ReadingListController和Spring Boot提供的BasicErrorController。

### 7.4.4 添加自定义trace repository

/trace接口默认使用容量为100的内存repository，我们可以自己声明InMemoryTraceRepository bean，比如设置容量为1000：

```
package readinglist;

import org.springframework.boot.actuate.trace.InMemoryTraceRepository;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ActuatorConfig {

  @Bean
  public InMemoryTraceRepository traceRepository() {
    InMemoryTraceRepository traceRepo = new InMemoryTraceRepository();
    traceRepo.setCapacity(1000);
    return traceRepo;
  }
  
}
```

虽然容量是增加了，但还是消耗内存，而且没有持久化，所以我们可以自己实现Spring Boot的TraceRepository接口：

```
package org.springframework.boot.actuate.trace;

import java.util.List;
import java.util.Map;

public interface TraceRepository {
  List<Trace> findAll();
  void add(Map<String, Object> traceInfo);
}
```

比如实现存到MongoDB：

```
package readinglist;
import java.util.Date;
import java.util.List;
import java.util.Map;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.trace.Trace;
import org.springframework.boot.actuate.trace.TraceRepository;
import org.springframework.data.mongodb.core.MongoOperations;
import org.springframework.stereotype.Service;

@Service
public class MongoTraceRepository implements TraceRepository {

  @Autowired
  private MongoOperations mongoOps;
  
  @Override
  public List<Trace> findAll() {
    return mongoOps.findAll(Trace.class);
  }
  
  @Override
  public void add(Map<String, Object> traceInfo) {
    mongoOps.save(new Trace(new Date(), traceInfo));
  }
}
```

你只要加入MongoDB的starter，就会自动配置好MongoOperations bean：

```
compile("org.springframework.boot:spring-boot-starter-data-mongodb")
```

### 7.4.5 自定义健康指标

比如我们要添加一个检测Amazon能否访问的健康指标：

```
package readinglist;
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

@Component
public class AmazonHealth implements HealthIndicator {

  @Override
  public Health health() {
    try {
      RestTemplate rest = new RestTemplate();
      rest.getForObject("http://www.amazon.com", String.class);
      return Health.up().build();
    } catch (Exception e) {
      return Health.down().build();
    }
  }
}
```

如果需要更详细的信息，可以调用withDetail()方法，比如把catch中的代码改成：

```
return Health.down().withDetail("reason", e.getMessage()).build();
```

## 7.5 Actuator接口安全

给Actuator路径加权限和一般的路径加权限是一样的，但是要用antMatchers把所有接口列一遍显然比较麻烦，所以可以配置一个公共的前缀：

```
management:
  context-path: /mgmt
```

然后就在SecurityConfig.java中的configure()方法中加入：

```
.antMatchers("/mgmt/**").access("hasRole('ADMIN')")
```