## Spring Boot 表单无法发送 PUT、DELETE 请求
put
 ```javascript
<form th:action="@{/emp}" th:method="POST">
  <!--SpringBoot的put请求方式-->
  <input type="hidden" name="_method" value="put" >
</form>
 ```

delete
```javascript
<form id="deleteForm" method="post">
  <!--SpringBoot的delete请求方式-->
  <input type="hidden" name="_method" value="delete"/>
</form>
```

无法发送 PUT 请求，原因是 springBoot 默认关闭了 MVC 的 HiddenHttpMethodFilter，因此需要打开。
打开方式：在 application.properties 中添加：
```yml
# 开启mvc的HiddenHttpMethodFilter，以便可以表单可以发送PUT、DELETE等请求
spring.mvc.hiddenmethod.filter.enabled=true
```

 
