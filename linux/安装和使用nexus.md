# 安装和使用maven私服 Nexus
## 安装
### 准备
[下载地址](http://www.sonatype.org/nexus/go/)
> 推荐这个地址[历史版本](http://www.sonatype.org/nexus/archived/)

### 安装
```
sudo tar -zxvf nexus-2.11.1-01-bundle.tar.gz -C /opt/nexus
sudo chown suncc /opt/nexus
sudo vi /etc/profile
> export PATH=/opt/nexus/nexus-2.11.1-01/bin:$PATH
source /etc/profile
```
### 启动
```
nexus start
```
访问 ：http://localhost:8081/nexus/#welcome

### 配置
#### 服务配置
```
/opt/nexus/nexus-2.11.1-01/conf/nexus.properties
```
#### 用户名密码
默认 ：u:admin p:admin123
```
/opt/nexus/sonatype-work/nexus/conf/security.xml
```

## 使用
[参考资料](http://aijezdm915.iteye.com/blog/1335025)

### 仓库类型
- hosted ：本地仓库，通常我们会部署自己的构件到这一类型的仓库
- proxy ：代理仓库，它们被用来代理远程的公共仓库，如maven中央仓库。
- group ： 仓库组，用来合并多个hosted/proxy仓库，通常我们配置maven依赖仓库组。
- virtual

### 配置
点击Central，并切换到Configuration选项卡

将Download Remote Indexes项设为True！这将打开nexus的下载远程索引的功能，便于使用nexus的搜索功能。

### 客户端mvn setting文件配置
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<settings xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository>D:\Program Files\apache-maven-3.2.3\repo</localRepository>
	<mirrors>
        <mirror>
            <id>osc</id>
            <mirrorOf>central</mirrorOf>
            <url>http://maven.oschina.net/content/groups/public/</url>
        </mirror>
    </mirrors>
    <profiles>
        <profile>
            <id>nexus</id>
            <!--Enable snapshots for the built in central repo to direct -->
            <!--all requests to nexus via the mirror -->
            <repositories>
                <repository>
                    <id>nexus-releases</id>
                    <url>http://nexus.vkelai.com:8081/nexus/content/repositories/releases/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
                <repository>
                    <id>nexus-snapshots</id>
                    <url>http://nexus.vkelai.com:8081/nexus/content/repositories/snapshots/</url>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
            </repositories>
        </profile>
    </profiles>
    <activeProfiles>
        <activeProfile>nexus</activeProfile>
    </activeProfiles>
    <proxies></proxies>
    <pluginGroups></pluginGroups>
</settings>

```