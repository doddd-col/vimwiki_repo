## 整合Shiro

**SecurityConfig**:配置
```java
@Configuration
public class ShiroConfig {

    /**
     * 1. 配置Realm (自定义类继承AuthorizingRealm)
     * 2. 配置SecurityManager
     * 3. 配置ShiroFilterFactoryBean
     */

    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean filterFactoryBean = new ShiroFilterFactoryBean();
        // 设置安全管理器
        filterFactoryBean.setSecurityManager(defaultWebSecurityManager);

        // 添加shiro的内置过滤器
        /**
         *  anon    无需验证
         *  authc   必须验证
         *  user    必须拥有记住我功能
         *  perms   拥有对某个资源的授权
         *  role    拥有角色权限
         */
		 
		 // 配置访问权限 必须是LinkedHashMap，因为它必须保证有序
        // 过滤链定义，从上向下顺序执行，一般将 /**放在最为下边 --> : 这是一个坑，一不小心代码就不好使了
        Map<String ,String> filterChainDefinitionMap = new LinkedHashMap<>();
       filterChainDefinitionMap.put("/add", "authc");
        filterChainDefinitionMap.put("/update", "authc");

        // 权限设置
        filterChainDefinitionMap.put("/add","perms[add]");
        filterChainDefinitionMap.put("/update","perms[update]");

        // 注销
        filterChainDefinitionMap.put("/logout", "logout");


        // 设置拦截
        filterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);

        // 设置登录页面
        filterFactoryBean.setLoginUrl("/toLogin");

        // 设置未授权页面
        filterFactoryBean.setUnauthorizedUrl("/noauthc");

        return filterFactoryBean;
    }

    @Bean(name = "securityManager")
    public DefaultWebSecurityManager defaultWebSecurityManager(@Qualifier("userRealm")UserRealm userRealm){
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        defaultWebSecurityManager.setRealm(userRealm);
        return defaultWebSecurityManager;
    }

    @Bean
    public UserRealm userRealm(){
        return new UserRealm();
    }
} 
```


**UserRealm**:认证和授权
```java
public class UserRealm extends AuthorizingRealm {

    @Autowired
    UserService userService;
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();

        Subject subject = SecurityUtils.getSubject();
        User currentUser = (User) subject.getPrincipal();
        simpleAuthorizationInfo.addStringPermission(currentUser.getPermission());
        return simpleAuthorizationInfo;
    }


    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        User user = userService.getUser(token.getUsername());

        if (user == null) {
            return null;
        }
        // 密码认证
        return new SimpleAuthenticationInfo(user,user.getPassword(),"");
    }
}

```


**Controller**  
```java
@Controller
public class MyController {

    @GetMapping("/")
    public String toLogin(Map<String, String> map) {
        map.put("msg", "helloworld");
        return "index";
    }

    @GetMapping("/add")
    public String add() {
        return "add";
    }

    @GetMapping("/update")
    public String update() {
        return "update";
    }

    @GetMapping("/toLogin")
    public String toLogin() {
        return "login";
    }

    @GetMapping("/login")
    public String login(String username, String password, Map<String, String> map) {
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken(username, password);
        try {
            subject.login(usernamePasswordToken);
            return "index";
        } catch (UnknownAccountException e) {
            map.put("msg", "用户名不存在");
            return "login";
        } catch (IncorrectCredentialsException e) {
            map.put("msg", "密码错误");
            return "login";
        }
    }

    @ResponseBody
    @GetMapping("/noauthc")
    public String noauthc(){
        return "未经授权";
    }
}
```



