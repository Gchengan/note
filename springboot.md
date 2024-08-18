# springboot邮件发送

```xml
<!--        邮箱依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
```





```yaml
  # QQ邮箱配置
mail:
  host: smtp.qq.com
  #自己的邮箱地址
  username: 2290601631@qq.com
  #授权码
  password: cbkfppkfdezyeafj
  # 编码格式
  default-encoding: utf-8
```


```java
@Autowired
private JavaMailSender javaMailSender;

SimpleMailMessage simpleMailMessage=new SimpleMailMessage();
//标
simpleMailMessage.setSubject("测试一下");
//发送人，这必须与我们刚刚的yml配置的邮箱地址要一样
simpleMailMessage.setFrom("2290601631@qq.com");
//接收人
simpleMailMessage.setTo("2290601631@qq.com");
///发送时间
simpleMailMessage.setSentDate(new Date());
//发送的邮件的具体内容
simpleMailMessage.setText("你好啊");
javaMailSender.send(simpleMailMessage);
```

# springboot整合shiro

##               不同角色登录认证：多个realm

### 1.创建enum类存储juese

```java
public enum RoleType {
    //管理员
    ADMIN,

    //员工
    EMPLOYEE
}
```



### 2.自定义UsernamePasswordToken，获取用户名，密码和角色信息

```java
public class UserToken extends UsernamePasswordToken {
  private RoleType roleType;


  public UserToken(String username,String password,RoleType roleType){
      super(username,password);
      this.roleType=roleType;
  }

    public RoleType getRoleType() {
        return roleType;
    }

    public void setRoleType(RoleType roleType) {
        this.roleType = roleType;
    }
}
```

### 3.继承ModularRealmAuthenticator，判断哪个角色该用哪个realms

```java
public class UserModularRealmAuthenticator extends ModularRealmAuthenticator {

    @Override
    protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
        //判断getReal是否为空
        assertRealmsConfigured();
        // 强制转换回自定义的CustomizedToken
        UserToken userToken = (UserToken) authenticationToken;
        //判断登录类型
        RoleType roleType = userToken.getRoleType();
        // 所有Realm
        Collection<Realm> realms = getRealms();
        // 登录类型对应的所有Realm
        Collection<Realm> typeRealms = new ArrayList<>();
        for (Realm realm : realms) {
            if (realm.getName().toLowerCase().contains(roleType.toString().toLowerCase())) {// 注：这里使用类名包含枚举，区分realm
                typeRealms.add(realm);
            }
        }
        // 判断是单Realm还是多Realm
        if (typeRealms.size() == 1) {
            return doSingleRealmAuthentication(typeRealms.iterator().next(), userToken);
        } else {
            return doMultiRealmAuthentication(typeRealms, userToken);
        }
    }
}
```

### 4.自定义不同角色的realm

<big>管理员admin</big>

```java
public class AdminRealm  extends AuthorizingRealm {

    @Autowired
    private AdminService adminService;

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        String username= (String) authenticationToken.getPrincipal();
        LambdaQueryWrapper<Admin> wr = new LambdaQueryWrapper<>();
        wr.eq(Admin::getUsername, wr);
        Admin admin = adminService.getOne(wr);
        if (admin==null){
            return null;
        }
        return new  SimpleAuthenticationInfo(username,admin.getPassword(),getName());
    }
}
```

员工

employee

```java
public class EmployeeRealm extends AuthorizingRealm {
    @Autowired
    private EmployeeService employeeService;

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
       String username = (String) authenticationToken.getPrincipal();
        LambdaQueryWrapper<Employee> wr = new LambdaQueryWrapper<>();
        wr.eq(Employee::getUsername, wr);
        Employee employee =employeeService.getOne(wr);
        if (employee==null){
            return null;
        }
        return new SimpleAuthenticationInfo(username,employee.getPassword(),getName());
    }
}
```

### 5.修改shiro的配置

```java
@Configuration
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
    //添加多个realm
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(AdminRealm adminRealm,EmployeeRealm employeeRealm,ModularRealmAuthenticator modularRealmAuthenticator){
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        //给安全管理器设置realm
        //设置多个realm
        defaultWebSecurityManager.setAuthenticator(modularRealmAuthenticator);
       //设置不同的realm
        ArrayList<Realm> realms = new ArrayList<>();
        realms.add(adminRealm);
        realms.add(employeeRealm);
        defaultWebSecurityManager.setRealms(realms);
        return defaultWebSecurityManager;
    }

    //创建自定义管理员Realm
    @Bean
    public AdminRealm adminRealm(){
        return new AdminRealm();
    }
    //创建自定义员工Realm
    @Bean
    public EmployeeRealm employeeRealm(){
        return new EmployeeRealm();
    }

    /**
     * 系统自带的Realm管理，主要针对多realm认证
     */
    @Bean
    public ModularRealmAuthenticator modularRealmAuthenticator() {
        //自己重写的ModularRealmAuthenticator
        UserModularRealmAuthenticator modularRealmAuthenticator = new UserModularRealmAuthenticator();
        modularRealmAuthenticator.setAuthenticationStrategy(new AtLeastOneSuccessfulStrategy());
        return modularRealmAuthenticator;
    }

}
```

### 6.修改UsernamPasswordToken为自己的UserToken

```java

    subject.login(new UserToken(admin.getUsername(), admin.getPassword(), RoleType.ADMIN));
```



# springboot整合redis

## 1.导入redis依赖

```xml
<!--        redis依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```



## 2.RedisTemplate操作

### 1.操作String类型数据

redisTemplate.opsForValue()，返回类型ValueOperations

### 2.操作Hash类型

redisTemplate.opsForHash，返回类型HashOperations

### 3.操作List类型

redisTemplate.opsForList，返回类型ListOperations

### 4.操作Set类型

redisTemplate.opsForSet，返回类型SetOperations

### 5.操作SortedSet类型

redisTemplate.opsForZSet，返回类型ZSetOperations

## 3.redis序列化

![屏幕截图 2023-02-03 225941](C:\Users\111\Desktop\配置文件\屏幕截图 2023-02-03 225941.png)

可以看到key和value都不是字符串,因为在使用java操作redis时，使用的是默认的jdk序列器

### 自定义RedisTemplate序列化方式

```java
@Configuration
public class RedisConfig {
    
    //连接工厂
    @Resource
    RedisConnectionFactory redisConnectionFactory;

    @Bean
    public RedisTemplate<String,Object> redisTemplate(){
        //创建RedisTemplate对象
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        //设置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        //创建JSON序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        //设置key的序列化,String类型
        redisTemplate.setKeySerializer(RedisSerializer.string());
        redisTemplate.setHashKeySerializer(RedisSerializer.string());
        //设置value
        redisTemplate.setValueSerializer(jsonRedisSerializer);
        redisTemplate.setHashValueSerializer(jsonRedisSerializer);
        return redisTemplate;
    }

}
```



# springboot整合jwt

## jwt

### jwt结构

#### 1.标头（Header）

**JWT头**是一个描述JWT元数据的JSON对象，alg属性表示签名使用的算法，默认为HMAC SHA256（写为HS256）；typ属性表示令牌的类型，JWT令牌统一写为JWT。最后，使用Base64 URL算法将上述JSON对象转换为字符串保存


```json
{  

"alg": "HS256", 

"typ": "JWT" 

}
```


#### 2.有效载荷（Payload）

**有效载荷**部分，是JWT的主体内容部分，也是一个**JSON对象**，包含需要传递的数据。 JWT指定七个默认字段供选择

iss：发行人
exp：到期时间
sub：主题
aud：用户
nbf：在此之前不可用
iat：发布时间
jti：JWT ID用于标识该JWT

这些预定义的字段并不要求强制使用。除以上默认字段外，我们还可以自定义私有字段，**一般会把包含用户信息的数据放到payload中**，如下例：

例如：

{"id":123,

"username":"admin",

"type":"admin"

}

请注意，**默认情况下JWT是未加密的，因为只是采用base64算法，拿到JWT字符串后可以转换回原本的JSON数据，任何人都可以解读其内容，因此不要构建隐私信息字段，比如用户的密码一定不能保存到JWT中**，以防止信息泄露。JWT只是适合在网络中传输一些非敏感的信息

#### 3.签名（Signature）

签名哈希部分是对上面两部分数据签名，需要使用base64编码后的header和payload数据，通过指定的算法生成哈希，以确保数据不会被篡改。首先，需要指定一个密钥（secret）。该密码仅仅为保存在服务器中，并且不能向用户公开。然后，使用header中指定的签名算法（默认情况下为HMAC SHA256）根据以下公式生成签名

HMACSHA256(base64UrlEncode(header)+"."+base64UrlEncode(payload),secret)



### jwt使用

#### 生成token

```java
    void contextLoads() {

        DateTime now = DateTime.now();
        DateTime newNow = now.offsetNew(DateField.MINUTE, 1);
        DateTime newNow_ = now.offsetNew(DateField.MINUTE, 2);
        String sign = JWT.create()
                .withClaim("userId", 1)
                .withClaim("username", "张三")
                //签发时间
                .withIssuedAt(now)
                //指定令牌的过期时间
                .withExpiresAt(newNow_)
                //有效时间
                .withNotBefore(newNow)
                .sign(Algorithm.HMAC256("sda15dw16a5"));
        System.out.println(sign);
    }
```

注意withNotBefore是指在此时间之前此token无效

#### 解析token

```java
    void get(){
        //解码
        JWTVerifier jwtVerifier = JWT.require(Algorithm.HMAC256("sda15dw16a5")).build();
        DecodedJWT verify = jwtVerifier.verify("eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NzU4NjA0MTEsInVzZXJJZCI6MSwidXNlcm5hbWUiOiLlvKDkuIkifQ.cW8DhVxDBz3qDjtjogKkSgyMxMTjGNOuOSfbgiT1DBM");
        System.out.println(verify.getClaim("username"));
        System.out.println(verify.getClaim("userId"));

    }
```






