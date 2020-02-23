默认是Tomcat

**jetty**  
```pom
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<exclusion>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<groupId>org.springframework.boot</groupId>
		</exclusion>
	</exclusions>
</dependency>

<dependency>
	<artifactId>spring-boot-starter-undertow</artifactId>
	<groupId>org.springframework.boot</groupId>
</dependency>
```

 

**undertow**  
```pom
<dependency>
	<artifactId>spring-boot-starter-undertow</artifactId>
	<groupId>org.springframework.boot</groupId>
</dependency>
```
