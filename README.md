# Spring security

> 스프링 시큐리티 쉽게 써보기, 이론 부분은 Reference 참고 후 개재

- [가이드 참고](https://spring.io/guides/gs/securing-web/)
- Reference 살펴보기

<br/>

## Getting Start

- 의존성 추가 (Thymeleaf는 이미 추가 했었음)

  ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
  </dependency>
  ```

- /, /hello 페이지 만들기

  ```html
  <!--home.html-->
  <!DOCTYPE html>
  <html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
      <head>
          <title>Spring Security Example</title>
      </head>
      <body>
          <h1>Welcome!</h1>
  
          <p>Click <a th:href="@{/hello}">here</a> to see a greeting.</p>
      </body>
  </html>
  <!--hello.html-->
  <!DOCTYPE html>
  <html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org"
        xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
      <head>
          <title>Hello World!</title>
      </head>
      <body>
          <h1>Hello world!</h1>
      </body>
  </html>
  ```

- View handler관련 Config파일 생성

  ```java
  @Configuration
  public class MvcConfig implements WebMvcConfigurer {
  
      public void addViewControllers(ViewControllerRegistry registry) {
          registry.addViewController("/home").setViewName("home");
          registry.addViewController("/").setViewName("home");
          registry.addViewController("/hello").setViewName("hello");
          registry.addViewController("/login").setViewName("login");
      }
  }
  ```

  - `WebMvcConfigurer`를 추가해 `addViewControllers()`를 상속 받는다.

- 이쯔음에서, login에 security를 적용하기위해 의존성을 하나 더 추가해준다.

  ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
  </dependency>
  ```

- Security handler 설정관련 Config파일 하나 더 생성

  ```java
  @Configuration
  @EnableWebSecurity
  public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
      @Override
      protected void configure(HttpSecurity http) throws Exception {
          http
              .authorizeRequests()
                  .antMatchers("/", "/home").permitAll()
                  .anyRequest().authenticated()
                  .and()
              .formLogin()
                  .loginPage("/login")
                  .permitAll()
                  .and()
              .logout()
                  .permitAll();
      }
  
    @Bean
      @Override
      public UserDetailsService userDetailsService() {
          UserDetails user =
               User.withDefaultPasswordEncoder()
                  .username("user")
                  .password("password")
                  .roles("USER")
                  .build();
  
          return new InMemoryUserDetailsManager(user);
      }
  }
  ```

  - `configure`메소드는 자주 쓸것이다. 여기에 다 설정하니깐..
  - `userDetailsService` 는 imemory에 테스트 해보려고 설정해놓음

- login 을 위한 view 생성

  ```html
  <!DOCTYPE html>
  <html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org"
        xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
      <head>
          <title>Spring Security Example </title>
      </head>
      <body>
          <div th:if="${param.error}">
              Invalid username and password.
          </div>
          <div th:if="${param.logout}">
              You have been logged out.
          </div>
          <form th:action="@{/login}" method="post">
              <div><label> User Name : <input type="text" name="username"/> </label></div>
              <div><label> Password: <input type="password" name="password"/> </label></div>
              <div><input type="submit" value="Sign In"/></div>
          </form>
      </body>
  </html>
  ```

- 그리고는 해당 유저 이름을 view로 띄우기 위해 hello.html 수정

  ```html
  <!DOCTYPE html>
  <html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org"
        xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
      <head>
          <title>Hello World!</title>
      </head>
      <body>
          <h1 th:inline="text">Hello [[${#httpServletRequest.remoteUser}]]!</h1>
          <form th:action="@{/logout}" method="post">
              <input type="submit" value="Sign Out"/>
          </form>
      </body>
  </html>
  ```
  - thymeleaf 기능으로 가져올 수 있다. `HttpServletRequest#getRemoteUSer()`

