# 7. Actuator

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
| GET | /info | Retrieves custom information about the application, as pro- vided by any properties prefixed with info. |
| GET | /mappings | Describes all URI paths and how they’re mapped to control- lers (including Actuator endpoints).
 |
