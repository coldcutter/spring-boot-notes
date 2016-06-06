# 8. 部署

## 8.1 部署到应用服务器

首先，我们构建一个war包：

```
apply plugin: 'war'

war {
    baseName = 'readinglist'
    version = '0.0.1-SNAPSHOT'
}
```

这样就能打成war包了，但目前这个war包没什么用，因为既没有包含web.xml也没有一个servlet initializer来enable Spring MVC的DispatcherServlet。