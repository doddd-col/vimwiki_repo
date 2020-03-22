## SpringSecurity简单整合
```java
/**
 * @program: spring-boot-springsecurity
 * @description:
 * @author: doddd
 * @create: 2020-03-03 21:30
 **/
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // Defining authorization rules
        //super.configure(http);
        http.authorizeRequests().antMatchers("/").permitAll()
                .antMatchers("/level1/**").hasAnyRole("VIP1")
                .antMatchers("/level2/**").hasAnyRole("VIP2")
                .antMatchers("/level3/**").hasAnyRole("VIP3");

        // enable automatic page generation
        http.formLogin();

        // enable automatic logout
        http.logout();
        // 访问/logout 表示注销并清空session

        // enable rememberMe
        http.rememberMe();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // Defining authentication rules
        //super.configure(auth);
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder()).withUser("zhangsan")
                .password(new BCryptPasswordEncoder().encode("123")).roles("VIP2")
                .and()
                .withUser("lisi").password(new BCryptPasswordEncoder().encode("345")).roles("VIP3");
    }
}
```

 
