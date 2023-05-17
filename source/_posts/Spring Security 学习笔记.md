---
title: Spring Security 学习笔记
tags: study
---
# 什么是 Spring Security ？

# ->项目依赖

```xml


- SpringSecurity
- jjwt-api
- jjwt-impl
- jjwt-jackson
- mysql
- mybatis
- lombok
- data-redis (暂定)

```

# ->  Spring Security 所需配置的部分

## 1. Spring Security 配置类（ SecurityConfig.java ）
 
 >  代码如下
 
 ```java

@Configuration  
@EnableWebSecurity  
@EnableMethodSecurity   // 精确到方法的权限控制
public class SecurityConfig {  

    @Autowired  
    // 自定义Jwt过滤器，在登录
    private JwtAuthFilter authFilter;  
  
    @Bean  
    // 自定义 UserDetailsService ，需要注册成Bean
    // 也可以在自定义的服务上添加@Component注解实现注解
    public UserDetailsService userDetailsService() {  
        return new UserInfoUserDetailsService();  
    }  
  
    @Bean  
    // 过滤器链设置： 重点
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {  
        return http
		        // 禁用CSRF
		        .csrf().disable()  
                .authorizeHttpRequests()  
		        // 放行登录注册请求
		        .requestMatchers("/products/new","/products/authenticate").permitAll()  
                .and()  
                // 设置其余请求均需要认证
                .authorizeHttpRequests().requestMatchers("/products/**")  
                .authenticated()
                .and()  
                // 设置Session管理为无状态
                .sessionManagement()  
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)  
                .and()  
                // 注册认证提供器，可以有多个，用于实现多种不同类型的登录
                .authenticationProvider(authenticationProvider())  
                // 添加自定义Jwt过滤器，位于UsernamePassword认证之前
                .addFilterBefore(authFilter, UsernamePasswordAuthenticationFilter.class)  
                .build();  
    }  
  
    @Bean  
    // 密码加密器，用于用户密码加密，Security推荐BCrypt
    public PasswordEncoder passwordEncoder() {  
        return new BCryptPasswordEncoder();  
    }  
  
    @Bean  
    // 注入AuthenticationProvider
    public AuthenticationProvider authenticationProvider(){  
        DaoAuthenticationProvider authenticationProvider=new DaoAuthenticationProvider();  
        authenticationProvider.setUserDetailsService(userDetailsService());  
        authenticationProvider.setPasswordEncoder(passwordEncoder());  
        return authenticationProvider;  
    }  
    @Bean  
    // 注入AuthenticationManager
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {  
        return config.getAuthenticationManager();  
    }  
  
}

```

## 2. JwtAuthFilter 

> 自定义过滤器实现提取token，并查询数据库进行认证
> 代码如下

```java

@Component  
public class JwtAuthFilter extends OncePerRequestFilter {  
  
    @Autowired  
    // JWT 工具类，用于token的生成和提取token中的信息
    private JwtService jwtService;  
  
    @Autowired  
    // 自定义的User信息提取类
    private UserInfoUserDetailsService userDetailsService;  
  
    @Override  
    // 过滤器主体
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {  
        // 提取请求头中的Token，有的话就验证并注册进Security的上下文中
        String authHeader = request.getHeader("Authorization");  
        String token = null;  
        String username = null;  
        if (authHeader != null && authHeader.startsWith("Bearer ")) {  
            token = authHeader.substring(7);  
            username = jwtService.extractUsername(token);  
        }  
  
        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {  
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);  
            if (jwtService.validateToken(token, userDetails)) {  
                UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());  
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));  
                SecurityContextHolder.getContext().setAuthentication(authToken);  
            }  
        }  
        filterChain.doFilter(request, response);  
    }  
}

```

## 3. JwtService Jwt工具类

> 代码如下

```java

@Component  
public class JwtService {  
  
    // Jwt加密密钥 ，原始Key -> Base64加密(256bit) -> 转16进制 -> 最终Key
    public static final String SECRET = "5367566B59703373367639792F423F4528482B4D6251655468576D5A71347437";  
  
    // 提取token中的用户名
    public String extractUsername(String token) {  
        return extractClaim(token, Claims::getSubject);  
    }  
    // 提取过期时间
    public Date extractExpiration(String token) {  
        return extractClaim(token, Claims::getExpiration);  
    }  
    // 从键值对中提取具体属性的键值对
    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {  
        final Claims claims = extractAllClaims(token);  
        return claimsResolver.apply(claims);  
    }  
    // 从token中的Payload中提取所有键值对
    private Claims extractAllClaims(String token) {  
        return Jwts  
                .parserBuilder()  
                .setSigningKey(getSignKey())  
                .build()  
                .parseClaimsJws(token)  
                .getBody();  
    }  
    // token过期验证
    private Boolean isTokenExpired(String token) {  
        return extractExpiration(token).before(new Date());  
    }  
    // token合法性验证，同时查看是否过期
    public Boolean validateToken(String token, UserDetails userDetails) {  
        final String username = extractUsername(token);  
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));  
    }  
  
    // Token生成
    public String generateToken(String userName){  
        Map<String,Object> claims=new HashMap<>();  
        return createToken(claims,userName);  
    }  
	// Token生成
    private String createToken(Map<String, Object> claims, String userName) {  
        return Jwts.builder()  
                .setClaims(claims)  
                .setSubject(userName)  
                .setIssuedAt(new Date(System.currentTimeMillis()))  
                .setExpiration(new Date(System.currentTimeMillis()+1000*60*30))  
                .signWith(getSignKey(), SignatureAlgorithm.HS256).compact();  
    }  
    // 解密出密钥
    private Key getSignKey() {  
        byte[] keyBytes= Decoders.BASE64.decode(SECRET);  
        return Keys.hmacShaKeyFor(keyBytes);  
    }  
}

```

## 4.  UserDetailsService
> 代码如下, 主要逻辑
> username -> UserMapper -> 查询出User(继承了UserDetails) 
> 												-> 查询到了 -> 返回
> 												-> 没查询到 -> 抛出异常

```java

@Component  
public class UserInfoUserDetailsService implements UserDetailsService {  
  
    @Autowired  
    private UserInfoRepository repository;  
  
    @Override  
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {  
        Optional<UserInfo> userInfo = repository.findByName(username);  
        return userInfo.map(UserInfoUserDetails::new)  
                .orElseThrow(() -> new UsernameNotFoundException("user not found " + username));  
  
    }  
}

```

## 5. Other
> 需要对方法进行权限验证时，在方法上添加 @PreAuthorize 注解即可，这需要在Security配置类上添加@EnableMethodSecurity 注解



## 6. 这里放一下UserDetails和正常User类之间的变换

```java

public class UserInfoUserDetails implements UserDetails {  
  
  
    private String name;  
    private String password;  
    private List<GrantedAuthority> authorities;  
  
    public UserInfoUserDetails(UserInfo userInfo) {  
        name=userInfo.getName();  
        password=userInfo.getPassword();  
        authorities= Arrays.stream(userInfo.getRoles().split(","))  
                .map(SimpleGrantedAuthority::new)  
                .collect(Collectors.toList());  
    }  
  
    @Override  
    public Collection<? extends GrantedAuthority> getAuthorities() {  
        return authorities;  
    }  
  
    @Override  
    public String getPassword() {  
        return password;  
    }  
  
    @Override  
    public String getUsername() {  
        return name;  
    }  
  
    @Override  
    public boolean isAccountNonExpired() {  
        return true;  
    }  
  
    @Override  
    public boolean isAccountNonLocked() {  
        return true;  
    }  
  
    @Override  
    public boolean isCredentialsNonExpired() {  
        return true;  
    }  
  
    @Override  
    public boolean isEnabled() {  
        return true;  
    }  
}

```

```java

public class UserInfo {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private int id;  
    private String name;  
    private String email;  
    private String password;  
    private String roles;  
}

```