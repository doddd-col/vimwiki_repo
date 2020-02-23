Circular view path [success]: would dispatch back to the current handler URL [/success] again. Check your ViewResolver setup! (Hint: This may be the result of an unspecified view, due to default view name generation.)

将修改版本的注释掉，即可正常运行
```pom
 	<properties>
		<java.version>11</java.version>
<!--		<thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>-->
<!--		<thymeleaf-layout-dialect.version>2.2.2</thymeleaf-layout-dialect.version>-->
	</properties>
```

 可能是版本依赖出了问题
