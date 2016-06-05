
# 3 自定义配置

## 3.1 覆盖Spring Boot的自动配置

我们将向reading-list项目中加入Spring Security，很简单，加入security starter即可：

```
compile("org.springframework.boot:spring-boot-starter-security")
```

然后你再运行项目，访问浏览器，就会有一个HTTP Basic认证的对话框，用户名填“user”，密码需要在启动日志里找，像这样的一行：

```
Using default security password: 297af950-707a-48d1-a2cb-cc29909080f3
```