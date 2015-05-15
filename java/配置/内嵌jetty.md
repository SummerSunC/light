# 项目内嵌jetty

---

## maven pom.xml
``` xml
<!-- jetty -->
<dependency>
	<groupId>org.eclipse.jetty.aggregate</groupId>
	<artifactId>jetty-webapp</artifactId>
	<version>${jetty.version}</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.eclipse.jetty</groupId>
	<artifactId>jetty-jsp</artifactId>
	<version>${jetty.version}</version>
	<scope>test</scope>
</dependency>
```
``` xml
<!--jetty插件-->
<build>
	<finalName>sun-snow-web</finalName>
	<plugins>
		<!--jetty插件-->
		<plugin>
			<groupId>org.mortbay.jetty</groupId>
			<artifactId>jetty-maven-plugin</artifactId>
			<version>${jetty.version}</version>
			<configuration>
				<systemProperties>
					<systemProperty>
						<name>spring.profiles.active</name>
						<value>development</value>
					</systemProperty>
				</systemProperties>
				<useTestClasspath>true</useTestClasspath>

				<webAppConfig>
					<contextPath>/${project.artifactId}</contextPath>
				</webAppConfig>
			</configuration>
		</plugin>
	</plugins>
</build>
```
## 启动项目
``` mvn
jetty:run
```







