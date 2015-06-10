# 安装和使用jenkins
## 准备
官方的已经很详细了

[官网](http://jenkins-ci.org/)

[官方安装教程](https://wiki.jenkins-ci.org/display/JENKINS/Use+Jenkins)

[官方启动安装教程](https://wiki.jenkins-ci.org/display/JENKINS/Starting+and+Accessing+Jenkins)


## 总结
```
COMMAND=/usr/bin/java -- -jar /home/jenkins/jenkins.war
```
jenkins 实际上是在内部嵌入了jetty容器，通过jar 的方式直接发布的，所以我们可以直接用jar命令直接运行war包就好了