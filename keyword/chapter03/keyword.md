# Q1. SOLID 원칙이란?

객체지향 설계의 5가지 핵심 원칙 ⇒ 더 좋은 소프트웨어를 만들기 위함

---

## 1) S - Single Responsibility Principle (단일 책임 원칙)

"하나의 클래스는 하나의 책임만 가져야 한다."

예를 들어, `UserService`에서는 사용자 관련 기능들만 관리해야 한다는 것이다.
`UserService`에서 Email, Auth, Review 등 다른 로직을 포함하지 않고 각각의 클래스로 분리해야 한다.

---

## 2) O - Open/Closed Principle (개방-폐쇄 원칙)

"확장에는 열려 있고, 수정에는 닫혀 있어야 한다."

새로운 기능을 추가할 때 기존 코드를 수정하지 않은 채로 개발을 진행할 수 있도록 설계해야 한다.
기존 코드를 직접 수정하지 않기 위해서 인터페이스 및 추상화 방식을 많이 사용한다.

> 💡 이번 워크북 중 DI, IoC는 '개방-폐쇄 원칙'을 지킨 기능이라고 볼 수 있다.

---

## 3) L - Liskov Substitution Principle (리스코프 치환 원칙)

"자식 클래스는 부모 클래스를 완전히 대체할 수 있어야 한다."

- 부모 클래스 = `Rectangle` (직사각형) / 자식 클래스 = `Square` (정사각형)
- 이때, 자식 클래스는 부모 클래스를 완전히 대체할 수 있는가? ❌
- 가로 5, 세로 3인 경우를 자식 클래스인 `Square`는 받을 수 없음

즉, LSP는 `"is-a"` 관계가 코드에서도 성립하는지 따져보는 것이 필요하다.

---

## 4) I - Interface Segregation Principle (인터페이스 분리 원칙)

인터페이스는 자신이 사용하지 않는 메서드를 포함하지 않도록 설계한다.

즉, 범용적으로 여러 기능을 가진 인터페이스가 아니라,
여러 개의 구체적이고 작은 범위의 인터페이스로 설계한다.

---

## 5) D - Dependency Inversion Principle (의존성 역전 원칙)

클래스가 의존 관계를 설정할 때, 구체 클래스가 아닌 추상 클래스(인터페이스)에 의존해야 한다.

예를 들어 `UserService`가 Repository를 의존할 때,
`JpaRepository`(구체 클래스)가 아닌 `Repository` 자체에 의존하도록 설정하고,
인터페이스를 통해 실제 객체를 주입받는 방식으로 설계해야 한다.

---

# Q2. DI란?

**의존성 주입(Dependency Injection)**이란 객체가 필요로 하는 의존 객체를 코드 내부에서 직접 생성하지 않고(`new`) 외부에서 생성하여 주입받는 방식이다.
주입 방법으로는 생성자 주입, Setter 주입, 필드 주입이 있으며, Spring에서는 생성자 주입을 권장한다.

Spring에서는 IoC 컨테이너가 Bean의 생성과 의존성 주입을 자동으로 관리하며,
개발자는 어노테이션(`@Component`, `@Service` 등)으로 Bean을 등록하면 컨테이너가 이를 수행한다.

생성자가 인터페이스 타입으로 의존성을 받기 때문에 다형성이 활용되며,
내부 코드 수정 없이 구현체를 교체할 수 있어 OCP를 준수할 수 있다.

---

# Q3. IoC란?

**IoC(Inversion of Control, 제어의 역전)**란 객체의 생성과 의존관계 설정의 제어권을
개발자가 아닌 프레임워크(Spring)가 갖는 설계 원칙이다.

전통적으로는 개발자가 `new`로 객체를 직접 생성하고 의존성도 직접 연결했지만,
IoC에서는 개발자가 어노테이션(`@Component`, `@Service` 등)으로 Bean을 선언만 하면
Spring의 IoC 컨테이너가 객체의 생성, 의존성 주입, 생명주기 관리를 자동으로 수행한다.

DI는 IoC를 구현하는 대표적인 방법으로, IoC 원칙을 실현하는 구체적인 방법이다.

> 💡 **IoC vs DI**
> - **IoC** : 제어권을 프레임워크에 넘기는 **원칙**
> - **DI** : 그 원칙을 실현하는 **구체적인 방법**

---

# Q4. 생성자 주입 vs 수정자 주입 vs 필드 주입

## 1) 필드 주입

`@Autowired`를 필드에 직접 선언하는 방식으로, 코드가 간결하지만
`final` 선언이 불가능하고 Spring 컨텍스트 없이는 의존성을 주입할 방법이 없어
순수 Java 단위 테스트가 어렵다.
또한 어떤 의존성이 필요한지 외부에서 파악하기 어려워 의존성이 숨겨진다는 단점이 있다.

## 2) Setter 주입

의존성을 선택적으로 주입할 수 있다는 점에서 유연하지만,
Setter 호출 없이도 객체가 생성되기 때문에 의존성이 누락된 채로 동작할 위험이 있다.
또한 외부에서 언제든 Setter를 호출해 의존성을 교체할 수 있어 불변성을 보장하지 못한다.

## 3) 생성자 주입 ✅

객체 생성 시점에 모든 의존성을 한 번에 주입하기 때문에
`final` 선언으로 불변성을 보장하고, 의존성이 누락되면 객체 자체가 생성되지 않아
필수 의존성을 강제할 수 있다.
또한 순환참조가 발생하면 애플리케이션 시작 시점에 즉시 감지되며,
Spring 없이도 순수 Java로 단위 테스트가 가능하다.
Spring이 생성자 주입을 공식적으로 권장하는 이유가 여기에 있다.

---

| | 생성자 주입 | Setter 주입 | 필드 주입 |
|---|---|---|---|
| `final` 선언 | ✅ | ❌ | ❌ |
| 순환참조 감지 | 시작 시점 | 런타임 | 런타임 |
| 단위 테스트 | ✅ 순수 Java | ✅ 가능 | ❌ Spring 필요 |
| 누락 의존성 | 생성 불가 | 런타임 NPE | 런타임 NPE |
| 의존성 명시성 | ✅ 명확 | ✅ 명확 | ❌ 숨겨짐 |
| 사용 권장 | ✅ 권장 | 선택적 의존성에만 | ❌ 지양 |

---

# Q5. AOP란?

**AOP(Aspect-Oriented Programming, 관점지향 프로그래밍)**란 트랜잭션 관리, 로깅, 인증 등
여러 객체에 공통적으로 사용되는 부가기능을 핵심 비즈니스 로직과 분리해 모듈화하는 프로그래밍 방법이다.
이를 통해 핵심 로직의 가독성을 높이고 중복 코드를 줄일 수 있다.

---

## AOP의 동작 원리

AOP는 프록시 패턴 기반으로 동작한다.
Spring 컨테이너가 `@Aspect`가 붙은 클래스를 감지해 대상 Bean의 프록시 객체를 생성하고,
메서드 호출 시 실제 객체 대신 프록시가 먼저 실행되어 부가기능(Advice)을 수행한 뒤 핵심 로직으로 위임한다.

주요 구성 요소로는 부가기능을 모듈화한 **Aspect**, 적용 대상 메서드를 지정하는 **Pointcut**,
실제 부가기능 코드인 **Advice**가 있다.
Advice는 실행 시점에 따라 `@Before`, `@After`, `@Around`로 구분된다.

- `@Before`: 메서드 실행 전
- `@After`: 메서드 실행 후
- `@Around`: 메서드 실행 전 후 모두 담당 (가장 많이 사용)

---

# Q6. 서블릿이란?

**Servlet**이란 Java EE(현재 Jakarta EE) 표준 스펙에 정의된, 프로토콜을 처리하기 위한 인터페이스이다.
본질적으로 Servlet은 웹 서버 ↔ Java Application 사이의 계약이다.
Java에서 "웹 요청을 처리하는 컴포넌트는 이 Servlet이라는 인터페이스를 구현해야 한다."고 표준화한 것이다.

---

## 왜 만들어졌는가?

웹 서버는 원래 정적인 HTML 파일만 반환할 수 있었다.
그러나 사용자마다 다른 응답을 줘야 하는 동적 처리(로그인, 마이페이지, 검색 결과 등)가 필요해졌고,
이를 위해 서블릿이 만들어졌다.

그리고 Servlet은 Servlet Container 없이 동작할 수 없다.

> **Servlet Container**란 실제 Servlet을 생성하고 관리하고 호출을 담당하는 컨테이너
```
TCP 소켓 연결
→ HTTP 요청 메시지 파싱
→ ServletRequest, ServletResponse 구현 객체 생성
→ URL 패턴에 맞는 Servlet 탐색
→ Servlet이 없으면 init() 호출로 인스턴스 생성 (최초 1회)
→ service() 호출
→ 응답 완료 후 소켓 처리
→ 애플리케이션 종료 시 destroy() 호출
→ TCP 연결 해제
```

---

## HttpServlet이란?

**HttpServlet**이란 Servlet 인터페이스를 HTTP에 특화해 구현한 추상 클래스이다.

### 상속 구조

1. `Servlet` (인터페이스)
2. `HttpServlet` (추상 클래스 - HTTP 특화)
3. `DispatcherServlet` (Spring MVC에서 사용)

HttpServlet의 핵심 역할은 `service()` 메서드 안에서 HTTP 메서드 (GET, POST, PUT, DELETE)를
분기해주는 것이다.
또한 `ServletRequest`, `ServletResponse` 대신 HTTP 전용 기능이 들어있는
`HttpServletRequest`, `HttpServletResponse`를 사용한다.

---

## DispatcherServlet이란?

**DispatcherServlet**이란 Spring이 만든 HttpServlet의 구현체이다.

핵심 역할은 **Front Controller 패턴의 구현**이다.
모든 URL 요청을 DispatcherServlet에서 한 번에 받고 내부적으로 라우팅하는 패턴이다.
기존의 방식은 요청 100개가 오면 Servlet 클래스를 100개 만들어야 했고,
공통 기능(인증 등)을 중복으로 전부 작성했어야 했다.

---

## 서블릿 동작 과정 - Spring MVC
```
클라이언트 HTTP 요청
→ Tomcat(서블릿 컨테이너) - TCP 연결 / HTTP 파싱 / Request, Response 객체 생성 후 전달
→ DispatcherServlet - Front Controller 패턴으로 모든 요청을 여기서 전부 받음
→ HandlerMapping - 요청 URL을 보고 어떤 Controller의 어떤 메서드에서 처리할지 결정
→ HandlerAdapter - Controller를 실제로 수행할 수 있는 형태로 변환하고,
                   @RequestParam, @RequestBody, @PathVariable 등의
                   파라미터 바인딩을 처리한 뒤 메서드를 호출한다.
→ Controller - 비즈니스 로직 수행
→ Service ↔ Repository ↔ DB
→ ViewResolver / MessageConverter
   - ViewResolver : HTML 템플릿 파일 찾아서 렌더링
   - MessageConverter : 객체를 JSON으로 직렬화
→ HTTP 응답 반환
```

> 💡 **핵심 포인트 : 관심사 분리**
> 개발자는 Controller 안의 비즈니스 로직만 개발하면 되고,
> 나머지 모든 과정은 Spring이 대신 처리해준다.