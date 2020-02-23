```java
 private void addErrorDetails(Map<String, Object> errorAttributes, WebRequest webRequest, boolean includeStackTrace) {
        Throwable error = this.getError(webRequest);
        if (error != null) {
            while(true) {
                if (!(error instanceof ServletException) || error.getCause() == null) {
				
					// 这里可以看到是添加exception的地方, 这有是当includeException为true是才能添加
                    if (this.includeException) {
                        errorAttributes.put("exception", error.getClass().getName());
                    }


                    this.addErrorMessage(errorAttributes, error);
                    if (includeStackTrace) {
                        this.addStackTrace(errorAttributes, error);
                    }
                    break;
                }

                error = error.getCause();
            }
        }

        Object message = this.getAttribute(webRequest, "javax.servlet.error.message");
        if ((!StringUtils.isEmpty(message) || errorAttributes.get("message") == null) && !(error instanceof BindingResult)) {
            errorAttributes.put("message", StringUtils.isEmpty(message) ? "No message available" : message);
        }

    }
```

```java

	默认无参构造器
    public DefaultErrorAttributes() {
        this(false);
    }
	
	有参构造器	includeException可以通过构造器传参
    public DefaultErrorAttributes(boolean includeException) {
        this.includeException = includeException;
    }
```

```java
	继承后通过调用父类构造器传参
    public MyErrorAttributes() {
        super(true);
    }
```

	
