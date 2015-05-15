# generatorConfig.xml

---
## pom.xml
``` xml
<plugins>
    <plugin>
      <groupId>org.mybatis.generator</groupId>
      <artifactId>mybatis-generator-maven-plugin</artifactId>
      <version>1.3.1</version>
      <configuration></configuration>
    </plugin>
</plugins>    
```
## generatorConfig.xml
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd" >
<generatorConfiguration>
    <!-- 这里要单独引入 -->
	<classPathEntry location="D:\Program Files\apache-maven-3.2.3\repo\mysql\mysql-connector-java\5.1.34\mysql-connector-java-5.1.34.jar" />
	<context id="context1">
		<commentGenerator>
			<property name="suppressDate" value="true" />
			<!-- 是否去除自动生成的注释 true：是 ： false:否 -->
			<property name="suppressAllComments" value="true" />
		</commentGenerator>

		<!--数据库链接URL，用户名、密码 -->
		<jdbcConnection driverClass="com.mysql.jdbc.Driver"
			connectionURL="jdbc:mysql://115.28.77.100:4407/fxt" userId="vcooline"
			password="123456" />
		<!-- 生成模型的包名和位置 -->
		<javaModelGenerator targetPackage="com.vcooline.fxt.common.model"
			targetProject="testMyBatisGenerator" />
		<!-- 生成映射文件的包名和位置 -->
		<sqlMapGenerator targetPackage="com.vcooline.wsite.dao.impl"
			targetProject="testMyBatisGenerator" />
		<!-- 生成DAO的包名和位置 -->
		<javaClientGenerator targetPackage="com.vcooline.fxt.common.mapper"
			targetProject="testMyBatisGenerator" type="XMLMAPPER" />

		<!-- 要生成哪些表 -->
		<table tableName="user" domainObjectName="User"
			enableCountByExample="false" enableUpdateByExample="false"
			enableDeleteByExample="false" enableSelectByExample="false"
			selectByExampleQueryId="false"></table>
	</context>
</generatorConfiguration>
```
## eclipse 安装插件