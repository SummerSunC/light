# 服务器tomcat配置

## linux配置
### jvm参数配置
```
vi $CATALINA_HOME/bin/catalina.sh

# 在文件头添加
JAVA_OPTS='-Xms512m -Xmx1024m …………'
```

### tomcat配置管理员用户
（非生产）
```
vi $CATALINA_HOME/conf/tomcat-user.xml

## 添加这里tomcat版本为 7或7以上 ,7以下版本不一样
  <role rolename="manager-gui"/>
  <role rolename="admin-gui"/>
  <user username="sunc" password="password" roles="manager-gui,admin-gui"/>
```
