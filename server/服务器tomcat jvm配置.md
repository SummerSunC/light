# 服务器tomcat配置

## linux 
### jvm参数配置
```
vi $CATALINA_HOME/bin/catalina.sh

# 添加
JAVA_OPTS='-Xms512m -Xmx1024m …………'
```

### tomcat配置管理员用户（version >= 7）
```
vi $CATALINA_HOME/conf/tomcat-user.xml

## 添加
  <role rolename="manager-gui"/>
  <role rolename="admin-gui"/>
  <user username="sunc" password="password" roles="manager-gui,admin-gui"/>
```