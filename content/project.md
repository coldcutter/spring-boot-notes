# 2 开发第一个应用

从本章开始，我们要开发一个简单的reading-list应用，用来维护一个reading-list，包括录入书的信息，查看阅读列表，删除书等操作。

技术上，Spring MVC处理Web请求，Thymeleaf作为模板引擎编写页面，Spring Data JPA操作数据库，使用内置H2数据库，用Gradle管理项目。

使用Spring Initializer生成

![Spring Initializer](QQ20160603-1.png)

生成的项目结构

![Project Structure](QQ20160603-2.png)

项目结构符合Gradle或者Maven的一般规范：

* 主代码位于/src/main/java
* 主资源位于/src/main/resources
* 测试代码位于/src/test/java
* 测试资源位于/src/test/resources

