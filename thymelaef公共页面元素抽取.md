## 公共元素抽取
 ```javascript
  抽取公共片段
  <div th:fragment="copy">
  </div>
  
  引入公共片段
  <div th:insert="~{footer :: copy}"></div>
  ~{templatename::selector} 模板名::选择器
  ~{templatename::fragmentname} 模板名::片段名
 ```

 insert片段默认在div标签里 
可以不用写~{}
行内写法[[~{}]] 要加上
 
 三种引入公共片段
1. th:insert
2. th:replace
3. th:include
```javascript
 <footer th:fragment="copy">
 </footer>
 
 <div th:insert="footer::copy"></div> :将公共片段直接插入到div中
 <div th:replace="footer::copy"></div> :将声明的元素替换为公共片段
 <div th:include="footer::copy"></div> :将被引入的元素包含到div中

效果：
 <div th:include="footer::copy">
	 <footer th:fragment="copy">
	 </footer>
 </div>
 
 
 <footer th:fragment="copy">
 </footer>


 <div th:include="footer::copy">
 </div>
```
