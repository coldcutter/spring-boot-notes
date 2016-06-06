# 8. 部署

## 8.1 部署到应用服务器

首先，我们构建一个WAR包：

```
apply plugin: 'war'

war {
    baseName = 'readinglist'
    version = '0.0.1-SNAPSHOT'
}

```