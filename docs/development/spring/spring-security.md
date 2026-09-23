---
title: Spring Security
---

## 1. 인증(Authentication)과 인가(Authorization)

- 인증(Authentication)
  - 시스템에 접근하려는 대상이 **'누구인지'** 확인하는 과정
  - 예를 들어, 아이디와 비밀번호를 통해 사용자가 정말 그 사람인지 검증하는 절차

- 인가(Authorization)
  - 인증된 사용자가 특정 리소스나 기능에 접근할 **'권한이 있는지'** 확인하는 과정
  - 예를 들어, 'ADMIN' 권한을 가진 사용자만 관리자 페이지에 접근할 수 있도록 허용하는 것

---

## 2. Spring Security 인증/인가 흐름

![Spring Security Authentication Flow](/assets/images/spring_security_flow.png)

- 스프링 시큐리티는 서블릿 컨테이너의 `Filter`를 기반으로 동작
- 하지만 서블릿 컨테이너와 스프링 컨테이너(IoC)는 서로 관리하는 영역이 다름 -> 연결 필요

#### ① DelegatingFilterProxy (서블릿 컨테이너 영역)

- 서블릿 필터와 스프링 빈(Bean)을 연결하는 다리(Proxy) 역할을 함
- 서블릿 컨테이너는 스프링이 관리하는 빈(Bean)들을 알지 못하므로 스프링 시큐리티의 보안 필터들을 직접 사용할 수 없음
- 서블릿 컨테이너에 이 프록시 필터를 등록해 두고, 요청이 오면 스프링 컨테이너에 있는 진짜 보안 필터에게 처리를 위임(Delegate)

#### ② FilterChainProxy (스프링 컨테이너 영역)

- `DelegatingFilterProxy`로부터 요청을 넘겨받아, 스프링 시큐리티가 관리하는 여러 `SecurityFilterChain` 중 어떤 것을 적용할지 결정하고 실행하는 역할을 함

#### ③ SecurityFilterChain (보안 필터 체인)

- 실제로 인증과 인가 처리를 담당하는 여러 개의 보안 필터(Security Filter)들이 사슬(Chain)처럼 엮여 있는 리스트
- 요청이 컨트롤러(DispatcherServlet)에 도달하기 전에 이 필터들을 순서대로 거치게 됨
- 하나라도 통과하지 못하면 요청은 거부됨

---

## 3. 핵심 컴포넌트

#### ① SecurityContextHolder & SecurityContext

- `SecurityContext`: 현재 인증된 사용자 정보(`Authentication`)를 보관하는 보관함
- `SecurityContextHolder`: 이 `SecurityContext`를 감싸고 있는 관리자
- 기본적으로 `ThreadLocal` 방식으로 동작하므로, 같은 쓰레드(하나의 요청 처리 쓰레드) 내에서는 어디서든 전역적으로 인증 정보를 조회할 수 있음

#### ② Authentication

- 시스템에 접근하는 대상이 누구인지 증명하는 인증 토큰(Token) 객체
- 인증 전에는 사용자가 입력한 아이디/비밀번호를 담고 있고, 인증이 완료되면 인증된 사용자의 정보와 권한 목록을 담게 됨

<details>
<summary>Authentication 상세</summary>
<div markdown="1">

```java
public interface Authentication extends Principal, Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    Object getCredentials();
    Object getDetails();
    Object getPrincipal();
    boolean isAuthenticated();
    void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

- `getPrincipal()`
  - 인증된 사용자의 식별자(Identity)를 반환
  - 보통 아이디/비밀번호 기반 로그인에서는 Spring Security의 `UserDetails` 구현체(사용자 정보 객체)가 들어있음

- `getAuthorities()`
  - 사용자에게 부여된 권한 목록(`ROLE_USER, ROLE_ADMIN`)을 반환하며 인가 과정에서 사용됨

- `getCredentials()`
  - 사용자가 본인임을 증명하기 위해 제출한 비밀번호나 토큰(자격 증명)을 반환
  - 인증이 완료(`AuthenticationManager`에 의해 처리)된 후에는 보안을 위해 보통 삭제(null 처리)됨

- `getDetails()`
  - 웹 요청과 관련된 추가적인 세부 정보를 반환
  - 주로 사용자의 IP 주소, 웹 세션 ID 등이 담긴 `WebAuthenticationDetails` 객체가 저장됨

- `isAuthenticated() / setAuthenticated(boolean)`
  - `isAuthenticated()`
    - 이 객체가 인증 완료된 사용자의 것인지(true), 아니면 이제 막 로그인을 시도하는 요청인지(false) 여부를 반환
  - `setAuthenticated(true/false)`
    - 인증 여부를 수동으로 설정
    - Spring Security 내부의 `AuthenticationManager`가 인증을 성공적으로 마친 후 true로 설정된 새로운 Authentication 객체를 만들 때 사용

> 즉, Authentication 객체는 어떤 사용자(Principal)가 어떤 증거(Credentials)를 가지고, 어디서(Details) 접근했으며, 검증 결과(isAuthenticated) 어떤 권한(Authorities)을 가졌는지를 모두 담고 있는 객체

</div>
</details>

#### ③ AuthenticationManager & ProviderManager

- 인증을 총괄하는 인터페이스
- 실제로 로그인을 처리할 때 `AuthenticationManager.authenticate(Authentication)` 메서드를 호출
- 가장 많이 사용되는 구현체는 `ProviderManager`
- `ProviderManager`는 직접 인증을 처리하기보다는, 자신이 가지고 있는 여러 `AuthenticationProvider`들에게 인증을 넘김

#### ④ AuthenticationProvider

- 실제 인증 로직(비밀번호 검증, 토큰 검증 등)을 수행
- 예를 들어, Form 로그인을 처리하는 `DaoAuthenticationProvider`는 입력된 비밀번호와 DB의 비밀번호가 일치하는지 검증
- 인증에 성공하면 사용자의 권한을 담은 완벽한 `Authentication` 객체를 만들어 반환

#### ⑤ UserDetailsService & UserDetails

- `UserDetails:` 스프링 시큐리티가 이해할 수 있는 형태의 사용자 정보 `VO(Value Object)` 인터페이스
- `UserDetailsService:` DB나 외부 시스템에서 사용자 정보를 조회해 오는 인터페이스
  - 단 하나의 메서드 `loadUserByUsername(String username)`만 가짐
  - DB의 회원 정보를 조회해 `UserDetails`로 리턴하면, `AuthenticationProvider`가 이 정보를 가지고 사용자가 입력한 비밀번호와 일치하는지 최종 검증을 수행
