# Spring Security OAuth2

> Spring Security 를 이용하여 OAuth2를 구현해보자

<br/>

## Getting Started

- 의존성 추가

  ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
  </dependency>
  
  <dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
    <version>2.3.5.RELEASE</version>
  </dependency>
  <dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
  </dependency>
  ```

  - 연습이니까 h2 db로 한다.
  - 구현시에는 다른걸 써야겠지?

- AuthorizationServer

  ```java
  @Configuration
  @EnableAuthorizationServer
  public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
  
      @Autowired
      private TokenStore tokenStore;
  
      @Autowired
      private AuthenticationManager authenticationManager;
  
      @Autowired
      private PasswordEncoder passwordEncoder;
  
      @Autowired
      private UserDetailsService userDetailsService;
  
      @Override
      public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
          clients
                  .inMemory()
                  .withClient("keesun-client")
                  .secret(passwordEncoder.encode("keesun-password"))
                  .authorizedGrantTypes("password",
                          "authorization_code",
                          "refresh_token",
                          "implicit")
                  .scopes("read", "write", "trust")
                  .accessTokenValiditySeconds(1*60*60)
                  .refreshTokenValiditySeconds(6*60*60);
  
      }
  
      @Override
      public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
          endpoints.tokenStore(tokenStore)
                  .authenticationManager(authenticationManager)
                  .userDetailsService(userDetailsService);
  
      }
  }
  ```

  - token 관련 서버 
    - Access token 및 refresh token 발급

- ResourceServer

  ```java
  @Configuration
  @EnableResourceServer
  public class ResourceConfig extends ResourceServerConfigurerAdapter {
  
      @Override
      public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
          resources.resourceId("resource_id").stateless(false);  
          // token만 사용하면 false, 다른 인증방법까지 사용하면 true
      }
  
      @Override
      public void configure(HttpSecurity http) throws Exception {
          http
                  .anonymous().disable()
                  .authorizeRequests()
                      .antMatchers("/users/**").authenticated()
                      .and()
                  .exceptionHandling()  //oauth2와 관련된 error 메세지 출력
                      .accessDeniedHandler(new OAuth2AccessDeniedHandler());
      }
  }
  ```

  - token을 통하여 REST API에 접근할 수 있게 인증해주는 resource 접근 인증 서버

- Security 관련 설정

  ```java
  //위의 두 Server에서 사용할 전반적인 설정
  
  @Configuration
  @EnableWebSecurity
  public class SecurityConfig extends WebSecurityConfigurerAdapter {
  
      @Resource(name = "userService")
      private UserDetailsService userDetailsService;
  
      @Bean
      public PasswordEncoder encoder() {
  
          return PasswordEncoderFactories.createDelegatingPasswordEncoder();
      }
  
      @Bean
      public TokenStore tokenStore() {
          return new InMemoryTokenStore();
      }
  
      @Bean
      @Override
      protected AuthenticationManager authenticationManager() throws Exception {
          return super.authenticationManager();
      }
  
      @Override
      protected void configure(AuthenticationManagerBuilder auth) throws Exception {
          auth.userDetailsService(userDetailsService)
                  .passwordEncoder(encoder());
      }
  
      @Override
      protected void configure(HttpSecurity http) throws Exception {
          http
                  .cors()
                      .and()
                  .csrf()
                      .disable()
                  .anonymous()
                      .disable()
                  .authorizeRequests()
                      .antMatchers("/api-docs/**").permitAll();
      }
      @Bean
      public CorsConfigurationSource corsConfigurationSource() {
  
          CorsConfiguration configuration = new CorsConfiguration();
          configuration.setAllowedOrigins(Arrays.asList("*"));
          configuration.setAllowedMethods(Arrays.asList("*"));
          configuration.setAllowedHeaders(Arrays.asList("*"));
          UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
          source.registerCorsConfiguration("/**", configuration);
          return source;
      }
  }
  
  ```

- 그 다음은 resource 서버에서 인증할, 설정해 놓은 서버 구현하면됌 (ex.user)

  - Model,controller,repository,service 등등..
  - service에서 중요점은 passwordEncoder 쓰는것
  - UserDetailsService를 상속받아, 구현하는것

<br/>

## Ref

- [가이드](https://www.devglan.com/spring-security/spring-boot-security-oauth2-example)
- [Youtube 백기선님 강의](https://www.youtube.com/watch?v=NQM1hghpF0Q&t=3172s)