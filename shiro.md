## shiro

Subject(用户):当前的操作用户 获取当前用户Subject currentUser = SecurityUtils.getSubject()
SecurityManager（安全管理器）:Shiro的核心,负责与其他组件进行交互,实现 subject 委托的各种功能
Realms（数据源） :Realm会查找相关数据源,充当与安全管理间的桥梁,经过Realm找到数据源进行认证,授权等操作
Authenticator（认证器): 用于认证，从 Realm 数据源取得数据之后进行执行认证流程处理。
Authorizer（授权器):用户访问控制授权，决定用户是否拥有执行指定操作的权限。
SessionManager （会话管理器）:支持会话管理
CacheManager （缓存管理器):用于缓存认证授权信息
Cryptography（加密组件）:提供了加密解密的工具包

![img](E:\studyMD\1)



 `public class ExcelController {​    /*    *导出数据    */​    @PostMapping("/export")    public R<String> getExportData(@RequestBody List<Employee> employee){     ExcelWriter writer= ExcelUtil.getWriter("e:/employee.xlsx");writer.addHeaderAlias("name", "姓名");writer.addHeaderAlias("sex", "性别");writer.addHeaderAlias("age", "年龄");writer.addHeaderAlias("phone", "手机号");writer.addHeaderAlias("username", "账号");writer.addHeaderAlias("email", "邮箱");writer.addHeaderAlias("address", "地址");// 默认的，未添加alias的属性也会写出，如果想只写出加了别名的字段，可以调用此方法排除之writer.setOnlyAlias(true);// 一次性写出内容，使用默认样式        writer.write(employee,true);// 关闭writer，释放内存        writer.close();return R.send("导出成功");    }​}java`
* : 匹配零个或多个字符串，如 /add* ,匹配 /addtest，但不匹配 /user/1

** : 匹配路径中的零个或多个路径，如 /user/** 将匹 配 /user/xxx 或 /user/xxx/yyy





| 配置缩写              | 对应的过滤器                         | 功能                                                                                                                  |
| ----------------- | ------------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| anno              | AnonymousFilter                | 指定url可以匿名访问                                                                                                         |
| authc             | FormAuthenticationFilter       | 指定url需要form表单登录，默认会从请求中获取账号密码等参数并尝试登录，如果登陆不了就会跳转到loginUrl配置的路径。我们也可以用这个过滤器做默认的登录逻辑，但是一般都是我们自己在控制器写登录逻辑，可以自定义返回出错的信息 |
| authcBasic        | BasicHttpAuthenticationFilter  | 指定url需要basic登录                                                                                                      |
| logout            | LogoutFilter                   | 登出过滤器，配置指定url就可以实现退出功能                                                                                              |
| noSessionCreation | NoSessionCreationFilter        | 禁止创建会话                                                                                                              |
| perms             | PermissionsAuthorizationFilter | 需要指定权限才能访问                                                                                                          |
| port              | PortFilter                     | 需要指定端口才能访问                                                                                                          |
| rest              | HttpMethodPermissionFilter     | 将http请求方法转化成相应的动词来构造一个权限字符串                                                                                         |
| roles             | RolesAuthorizationFilter       | 需要指定角色才能访问                                                                                                          |
| ssl               | SslFilter                      | 需要https请求才能访问                                                                                                       |
| user              | UserFilter                     | 需要已登录或者记住我的用户才能访问                                                                                                   |



## 自定义登录逻辑，改变默认跳转页面方式，而是返回json信息

```java
public class LoginFilter extends FormAuthenticationFilter {

    private static final  Logger log=LoggerFactory.getLogger(FormAuthenticationFilter.class);

    /**
     * 在访问controller前判断是否登录，返回401，不进行重定向。
     *
     * @return true-继续往下执行，false-该filter过滤器已经处理，不继续执行其他过滤器
     */
    @Override
    protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {

        if (this.isLoginRequest(request, response)) {
            if (this.isLoginSubmission(request, response)) {
                if (log.isTraceEnabled()) {
             log.trace("Login submission detected.  Attempting to execute login.");
                }

                return this.executeLogin(request, response);
            } else {
                if (log.isTraceEnabled()) {
                    log.trace("Login page view.");
                }

                return true;
            }
        } else {
            if (log.isTraceEnabled()) {
                log.trace("Attempting to access a path which requires authentication.  Forwarding to the Authentication url [" + this.getLoginUrl() + "]");
            }
            response.setContentType("text/json;charset=UTF-8");
            response.getWriter().println(JSON.toJSONString(R.send("请登录")));
            return false;
        }
    }
}
```



## shiro配置与使用

```java
public class ShiroConfig {

    //创建shiroFilter拦截请求
    @Bean(name = "shiroFilterFactoryBean")
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //给filter设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(defaultWebSecurityManager);
        //自定义过滤器
        Map<String, Filter> filterMap=new HashMap<>();
        filterMap.put("authc", new LoginFilter());
        //配置受限资源
        Map<String,String> map=new HashMap<>();
        map.put("/employee/**","authc");//authc表示这个请求需要认证和授权
        shiroFilterFactoryBean.setFilters(filterMap);
        shiroFilterFactoryBean.setFilterChainDefinitionMap(map);
        return shiroFilterFactoryBean;
    }

    //创建安全管理器
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(Realm realm){
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        //给安全管理器设置realm
        defaultWebSecurityManager.setRealm(realm);
        return defaultWebSecurityManager;
    }

    //创建自定义realm
    @Bean
    public Realm getRealm(){
        return new RealmConfig();
    }

}
```

## shiro认证与授权

例子

```java
public class LoginController {


    //管理员
    @PostMapping("/admin")
    public R<String> login(@RequestBody Admin admin) {

        Subject subject = SecurityUtils.getSubject();
        System.out.println(subject);
        try {
            subject.login(new
    UsernamePasswordToken(admin.getUsername(),admin.getPassword()));
        } catch (AuthenticationException e) {
           if (e instanceof UnknownAccountException||e instanceof IncorrectCredentialsException){
               return R.send("用户名或密码错误");
           }else if (e instanceof LockedAccountException){
               return R.send("账户被锁定");
           }else {
               return R.send("未知错误");
           }
        }

        return R.send("登录成功");
    }

}
```

注意：当返回null时，才是UnknownAccountException异常

```java
public class RealmConfig extends AuthorizingRealm {

    @Autowired
    private AdminService adminService;

    //处理授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }


    //处理认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        String principal = (String) authenticationToken.getPrincipal();
        LambdaQueryWrapper<Admin> wr = new LambdaQueryWrapper<>();
        wr.eq(Admin::getUsername,principal);
        Admin admin = adminService.getOne(wr);
          if (admin==null){
            return null;
        }
        SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(principal, admin.getPassword(),this.getName() );
        return simpleAuthenticationInfo;
    }
}
```
