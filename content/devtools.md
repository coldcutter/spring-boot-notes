# A. 开发者工具

Spring Boot 1.3引入了一系列新的开发者工具，在开发阶段，可以利用它们完成：

* 自动重启——当classpath下的文件改变的时候可以在运行中自动重启
* LiveReload——改变资源会触发浏览器自动刷新
* 远程开发——远程部署支持自动重启和LiveReload
* 默认开发属性——提供某些配置属性的明智的开发时默认值

要启用开发者工具，只需要引入starter就好了：

```
compile "org.springframework.boot:spring-boot-devtools"
```

当你的应用是以完整的JAR包或WAR包运行时，开发者工具会被禁用，所以构建生产部署包的时候没有必要删掉这个依赖。

**自动重启**

当devtools生效时，classpath下的文件一修改就会触发自动重启。为了使重启尽可能快，那些不会改变的类（比如第三方JAR包中的类）会被加载到一个base classloader，而应用代码会被加载到一个分开的restart classloader。当检测到改变时，只有restart classloader会重启。

有些classpath下的资源的改变不用重启应用，比如Thymeleaf模板，还有/static或/public下的静态资源，所以devtools重启功能不会考虑如下路径：/META-INF/resources，/resources，/static，/public，/templates。不过这可以通过spring.devtools.restart.exclude设置：

```
spring:
  devtools:
    restart:
      exclude: /static/**,/templates/**
```

当然你也可以完全禁用自动重启：

```
spring:
  devtools:
    restart:
      enabled: false
```

另一个选择是设置一个触发文件，只有当这个触发文件改变了才会重启，比如当且仅当.trigger文件修改时才重启：

```
spring:
  devtools:
    restart:
      trigger-file: .trigger
```

如果IDE持续编译修改的文件，那么触发文件就很有用，没有触发文件，每次修改都会重启，有了触发文件，只有在需要的时候会重启（通过修改触发文件）。

**LiveReload**

一般的Web开发过程都需要每次修改页面、js或css，然后刷新浏览器来看效果，Spring Boot devtools集成了[LiveReload](http://livereload.com)，就不需要刷新了。Spring Boot会启动一个内置的LiveReload server，当资源修改的时候会触发浏览器刷新，当然你需要在浏览器中安装LiveReload插件。

你也可以禁用LiveReload server：

```
spring:
  devtools:
    livereload:
      enabled: false
```

**远程开发**

devtools支持远程调试，一般情况下，你不会用到远程调试，比如生产环境，但当你需要用一些本地没法使用的云服务时，可能会有一个远程开发环境，首先你得设置一个远程密码：

```
spring:
  devtools:
    remote:
      secret: myappsecret
```

当你设置了这个属性，会启用一个支持远程开发的组件，它会监听进来的修改请求，重启应用或者触发浏览器刷新。

然后，你得在本地运行远程开发客户端，以类的形式，类名是org.springframework.boot.devtools.RemoteSpringApplication，被设计成在IDE里面运行，参数是远程应用的地址。

假如你把reading-list部署在Cloud Foundry上，地址是 https://readinglist.cfapps.io 。比如你使用IntelliJ IDEA，如下步骤启动client：

1. 选择Run > Edit Configurations...
2. 按“+”号，选择“Application”
3. 在Main class中填写org.springframework.boot.devtools.RemoteSpringApplication
4. 在Program arguments中填写https://readinglist.cfapps.io

确定，然后就可以运行client了，你就可以修改你IDE中的应用了，当检测到修改时，它们会被推送到远程server。

你还可以在IDE中远程调试，不过你得确保远程应用启用了远程调试，通常可以通过配置JAVA_OPTS来启用，举个例子，如果你的应用部署在Cloud Foundry上，你可以在你的应用的manifest.yml文件里设置：

```
---
  env:
    JAVA_OPTS: "-Xdebug -Xrunjdwp:server=y,transport=dt_socket,suspend=n"
```

当远程应用启动且与本地调试服务器建立了连接后，你就可以像调试本地应用一样设置断点和单步调试了（因为网络延迟可能会有点慢）。

**默认开发属性**

有些配置属性通常在开发的时候都需要设置，但在生产配置中不用，比如视图模板缓存，在开发中最好关闭，这样修改模板可以立刻看到效果，但是为了性能，在生产上应该启用。

Spring Boot默认对所有支持的模板启用缓存，但是如果用了开发者工具，缓存会被禁用，本质上就是下面的属性会被设置成false：

* spring.thymeleaf.cache
* spring.freemarker.cache
* spring.velocity.cache
* spring.mustache.cache
* spring.groovy.template.cache

这样你就省得在开发的时候自己禁用它们了。

**全局配置开发者工具**

在你的用户目录下创建一个文件.spring-boot-devtools.properties（注意有一个点），在这个文件里你可以设置所有项目共用的开发者工具属性，比如：

```
spring.devtools.restart.trigger-file=.trigger
spring.devtools.livereload.enabled=false
```

然后，如果要覆盖这些属性，在你的项目里配置就行了。