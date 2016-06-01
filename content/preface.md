# 前言


## Spring发展历史

* Spring 1.0，它改变了企业级Java应用开发。依赖注入和声明式事务意味着组件之间不再需要紧耦合，不再需要笨重的EJBs。
* Spring 2.0，自定义XML命名空间，更小、更容易理解的配置文件。
* Spring 2.5，提供了优雅的面向注解的依赖注入模型，@Component和@Autowired，以及Spring MVC编程模型。无需显式声明组件，无需继承一些基础控制器类。
* Spring 3.0，全新的基于Java的配置方式，替换XML，从Spring 3.1开始的@Enable打头的注解。使得写一个零XML的Spring应用更加现实。
* Spring 4.0，条件配置，运行过程中可以根据类路径，环境等因素觉得哪些配置生效，哪些配置忽略。不需要在构建的时候写脚本来决定打包哪些配置了。

## Spring Boot

Spring Boot提供自动配置，可以自动配置组件，在大部分情况下不再需要显式的配置。

Spring Boot有很多starter依赖包，这些starter包聚集了许多依赖包，你只需要引入这些starter包，几乎所有需要用的第三方包都会引入进来（通过依赖传递），而且这些starter包的依赖版本都是经过严格测试的，保证版本兼容性不会出现问题。

Spring Boot CLI（command-line interface）通过它可以用Groovy开发应用程序，省了一大坨代码，Getter和Setter、访问修饰符public、private，分号，return关键字这些都不用写，并且多数情况下import语句都不用写，而且你是通过命令行跑脚本的，所以都不用build。

Spring Boot Actuator，通过它你可以看到运行中的应用程序的很多信息，有哪些bean以及它们的依赖关系，控制器路径映射，配置属性、环境变量等等。

Spring Boot并非一种全新的框架，而是在Spring的基础之上，提供了开发Spring应用程序的更便捷的方法。