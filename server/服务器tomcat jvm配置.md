# 服务器tomcat配置

## linux 
### jvm参数配置
```
vi $CATALINA_HOME/bin/catalina.sh

# 添加
JAVA_OPTS='-Xms512m -Xmx1024m …………'
```

### tomcat配置管理员用户

```
vi $CATALINA_HOME/conf/tomcat-user.xml


## 添加
# 这里tomcat版本为>= 7 ,7以下版本不一样
  <role rolename="manager-gui"/>
  <role rolename="admin-gui"/>
  <user username="sunc" password="password" roles="manager-gui,admin-gui"/>
```