# 安装maven私服 Nexus

## 准备
[下载地址](http://www.sonatype.org/nexus/go/)
> 推荐这个地址[历史版本](http://www.sonatype.org/nexus/archived/)

## 安装
```
sudo tar -zxvf nexus-2.11.1-01-bundle.tar.gz -C /opt/nexus
sudo chown suncc /opt/nexus
sudo vi /etc/profile
> export PATH=/opt/nexus/nexus-2.11.1-01/bin:$PATH
source /etc/profile
```
## 启动
```
nexus start
```
访问 ：http://localhost:8081/nexus/#welcome