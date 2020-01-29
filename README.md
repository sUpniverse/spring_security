# 스프링 시큐리티

목차

- 개요
- 설정 방법
  - OAuth2
- 출처

## 개요

스프링 시큐리티

- 웹 시큐리티 (Filter 기반 시큐리티)

  - 웹 요청이 들어왔을때
  - Servlet, WebFlux 관련 2개가 있지만, 아래 설명은 Servlet 관련된 설명

- 메소드 시큐리티 

  - 해당 메소드가 호출되었을 때

- 이 둘 다 Security Interceptor를 사용한다.

  - 리소스에 접근을 허용할 것이냐 말것이냐를 결정하는 로직이 들어있음.

    ![img](https://lh6.googleusercontent.com/ZGYaju6tHzRxHWc8752zZKsczHYvJtba-suer56bEJhr0SgspP9SGBOnVxzkhE5HAWm1SraOX8-E2H5IaDnBznbwq1D_uj7mUR3Tu-iQwa-M2Roy4r_uT0jHpJEulymEzjmxwIZA)

    - `SecurityContextHolder`
      - Java `ThreadLocal`
        - 한 쓰레드 내에서 데이터를 공유하는 자원
        - 파라미터를 이용해서 다른 메소드에 정보를 전달이 아닌, ThreadLocal에 저장해놓고 공유한다.
      - 다른 구현체로 바꿀 수도 있음

  - 과정

    - `SecurityContextHolder`에서 인증정보를 꺼낸다.
      - 없으면 인증되지 않은 사용자
    - `AuthenticationManager`를 통해 로그인 (인증)
      - 여러가지 로그인 방법이 있음
        - Basic
        - JWT
        - ...
      - `UserDetailsServie`
        - Interface를 사용해 입력받은 User정보를 이용해 DB에서 Password를 가져옴
      - `PasswordEncoder`
        - `UserDetailsServie`를 통해 얻어온 실제 PW와 입력된 PW를 비교해 맞는지 확인
    - `AccessDecisionManager` (인가)
      - 현재 인증된 Account의 Role이 해당 기능을 사용할 수 있는지 확인
      - ...

아키텍쳐 및 과정

<img src="https://chathurangat.files.wordpress.com/2017/08/blogpost-spring-security-architecture.png">

1. **Http Request를 받는다**

   - 스프링 시큐리티는 filter chain으로 구성되어있다.
   - Request가 들어올때, 인증/인가를 위하여 해당 필터를 지나가게 된다.

2. **유저의 신원증명을 기반으로한 인증토큰을 생성한다**

   - Form을 통해 Id 및 password가 들어오면 AuthenticationFilter가 해당 정보를 인터셉트한다.
   - 그 후에 로그인시도 유저의 정보를 이용한 인증객체를 생성하게 된다. (실제 인증을 한것이 아니다.)

3. **AuthenticationManager에게 인증 토큰을 위임한다**

   - `UsernamePasswordAuthenticationToken` 객체를 생성한 후 해당 정보를 Manager에게 넘긴다.
     - `AuthenticationManager`의 `authenticate` 메소드를 호출하여 해당 인증 객체를 넘겨준다.
   - 여기서 `AuthenticationManager` 는 인터페이스로서 실제의 구현은  `ProviderManager`가 하고있다.
     - `ProviderManager` 는 유저를 인증하기 위해 설정된 `AuthenticationProvider`들을 가지고 있다.
     -  `AuthenticationProvider` 들은 유저기반의 인증 요청을 위임받아 인증하게 된다.

4. **AuthenticationProvider들을 이용해 인증을 시도한다**

   - 제공된 인증객체를 이용해 유저를 인증하게 된다.
   - 이때, 사용되는 Provider의 종류는 아래와 같다.
     - `CasAuthenticationProvider`
     - `JaasAuthenticationProvider`
     - `DaoAuthenticationProvider`
     - `OpenIDAuthenticationProvider`
     - `RememberMeAuthenticationProvider`
     - `LdapAuthenticationProvider`

5. **UserDetailsService가 필요할까?**

   - 몇몇 AuthenticationProvider은 아마도 유저의 정보(유저의 이름 기반)를 읽어오기 위해 **UserDetailsService**를 사용할 것이다.
     - RestAPI에서 보안을 적용할때도, 해당 DetailsService를 UserService에 구현했었다.

6. 7. **UserDetails 혹은 User Object?**

   - UserDetailsService는 username을 기반으로한 UserDetails를 읽어올 것이다.
   - `loadUserByUsername(String username)`을 호출하여 UserDetails 객체를 반환한다.
     - UserDetails로의 반환은 스프링 시큐리티가 이해할 수 있는 타입으로 변환해줘야 하기 때문
     - 해당 UserDetails의 타입 중 스프링 시큐리티가 제공하는 User 객체를 만들어 반환하면 된다.

8. **인증객체 혹은 인증에러**
   - 유저가 성공적으로 인증된다면, 완전한 정보를 가진 인증 객체가 반환될 것이고, 그렇지 않다면 인증에러가 던져질 것이다.
     - 완전한 정보를 가진 인증객체란?
       - 인증정보 : 참
       - 인가 정보 : 역할
       - 유저의 정보 : loadUserByUsername에 쓰였던 인증 정보만 ex: 유저이름 혹은 유저아이디, 메일등
   - 만약 인증에러가 던져지면 인증 구조에 의해 해당 에러가 `AuthenticationEntryPoint`에 의해 처리된다.
9. **인증 완료!**
   - AuthenticationManager는 완전한 정보를 가진 인증객체를 Authentication Filter에 반환할 것이다.
10. **SecurityContext에 인증객체를 담는다**
    - 반환된 인증 객체는 SecurityContext에 담아 놓고 인증과정에서 사용한다. (위의 SecurityContextHolder 참조)
      - 담은 이후에 성공시에는 AuthenticationSuccessHandler를 실행한다.
      - 실패시에는  AuthenticationFailureHandler를 실행하게 된다.

## 출처

- 백기선님의 [REST API](https://www.inflearn.com/course/spring_rest-api) 강좌 섹션 중 REST API 보안적용
- [SpringBootDev](https://springbootdev.com/2017/08/23/spring-security-authentication-architecture/)

