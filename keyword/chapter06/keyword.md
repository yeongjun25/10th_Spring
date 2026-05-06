# Q1. JPA란?

**JPA**(Java Persistence API)는 자바 진영에서 정의한 **ORM 표준 스펙**이다.

**ORM**(Object Relational Mapping)은 객체(Java 클래스)와 관계형 DB의 테이블을 1:1로 매핑하는 기술이다. JPA는 구현체가 아닌 인터페이스이며, JPA의 구현체가 바로 **Hibernate**이다. (가장 많이 사용)

```java
@Entity
@Table(name = "members")    // @Table 어노테이션이 핵심
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String name;
}
```

이렇게 `@Table` 어노테이션만 붙이면 `members` 테이블과 `Member` 객체가 매핑이 된다.

## 왜 사용하는가?

### 1. 생산성 향상

**핵심**: SQL을 작성하지 않고 그냥 객체만 다루면 된다. (객체 중심 개발)

```
// 객체 생성 (Create)
Member member = new Member("Jay");
em.persist(member);                      // INSERT 자동 생성

// 객체 조회 (Read)
Member jay = em.find(Member.class, 1L);  // SELECT 자동 생성 

// 객체 수정 (Update)
jay.setName("Jai");                      // Update 자동 호출. setter로 사용

// 객체 삭제 (Delete)
em.remove(jay);                          // Delete 자동 생성
```

여기서 특히 Update의 **'변경 감지'**가 핵심이다. 객체 정보를 수정 후 SQL을 따로 날리는 것이 아니라, 트랜잭션이 커밋되는 순간에 JPA가 변경된 부분만 자동으로 감지해서 UPDATE 쿼리를 날린다.

### 2. 유지보수 쉬움

**핵심**: 개발 도중 필드 추가/변경 시 SQL을 일일이 수정할 필요가 없다.

JDBC의 경우 필드를 변경했을 때, 관련된 SQL을 전부 수정해줘야 한다. 반면 JPA는 알아서 SQL을 다시 생성해준다. 이는 단순히 편리함 제공을 넘어, **버그 발생률을 크게 감소**시킨다.

### 3. 성능 최적화

- **1차 캐시**: 같은 트랜잭션 내에서 같은 PK로 조회를 여러 번 할 때, 두 번째 접근은 SQL을 날리지 않고 1차 캐시에서 데이터를 가져온다.

```
  Member m1 = em.find(Member.class, 1L);  // SELECT 쿼리 발생
  Member m2 = em.find(Member.class, 1L);  // SELECT 쿼리 발생 X, 캐시에서 반환
  
  System.out.println(m1 == m2);  // m1 == m2 동일성 보장까지 가능함
```

- **쓰기 지연**: 한 트랜잭션 내에서 발생한 INSERT/UPDATE/DELETE는 커밋 시점에 한 번에 진행된다. 즉, 커밋 시점 전까지는 `persist`, `set`, `remove`를 실행해도 즉각 반영되지 않는다.
    - `persist`: 쓰기 지연 저장소에 INSERT 쿼리가 쌓인다.
    - `remove`: 쓰기 지연 저장소에 DELETE 쿼리가 쌓인다.
    - `find`: 즉각 수행된다. (이때의 상태를 스냅샷으로 복사하여 저장해 놓음)
    - `set`: 커밋 시점에 스냅샷과 비교하여 변경된 부분이 있으면 쓰기 지연 저장소에 쿼리가 쌓인다.

  **장점**: 네트워크 왕복 횟수가 줄어든다 ⇒ 성능 효율

- **변경 감지**: 커밋 시점에 JPA는 엔티티의 스냅샷(원본)과 현재 상태를 비교하여 변경된 부분을 감지하고, 변경이 있으면 UPDATE한다.

- **지연 로딩 / 즉시 로딩 선택**: 연관 객체를 꼭 필요할 때만 가져오도록 늦출 수 있다.  
  `@ManyToOne(fetch = FetchType.LAZY)`, fetch join 등

---
# Q2. N+1 문제란?

N개의 결과를 가져오는 쿼리를 1번 날렸을 뿐인데, 연관 데이터를 조회하느라 추가로 N번의 쿼리가 발생하는 현상. 총 **1+N개의 쿼리**가 발생하므로 N+1 문제라고 부른다.

- 일대다 관계(1:N)의 엔티티를 조회할 때 발생한다.
- **즉시 로딩**인 경우 발생한다. (JPA fetch 전략이 `EAGER`인 경우)
    1. JPQL에서 만든 SQL을 통해 데이터를 조회
    2. JPA에서 Fetch 전략(`EAGER`)으로 해당 데이터의 연관 관계인 하위 엔티티들을 추가 조회
    3. 2번 과정으로 N+1 문제 발생
- **지연 로딩**인 경우 발생한다. (JPA fetch 전략이 `LAZY`인 경우)
    1. JPQL에서 만든 SQL을 통해 데이터를 조회
    2. JPA에서 Fetch 전략(`LAZY`)으로 조회하지만, 지연 로딩이기 때문에 추가 조회는 하지 않음
    3. 하지만, 하위 엔티티에 접근하여 작업하게 되면 추가 조회가 발생하기 때문에 결국 N+1 문제 발생

### [N+1 해결 방법]

근본적으로 N+1 문제가 발생하는 원인은 한쪽 테이블만 조회를 하고 연결된 다른 테이블은 따로 조회하기 때문이다. 따라서, 미리 두 테이블을 Join하여 한 번에 모든 데이터를 가져오면 N+1 문제가 발생하지 않는다.

### 1. Fetch Join

쿼리가 1번만 발생하고 미리 두 테이블을 Inner Join하여 데이터를 한 번에 가져온다.

```
@Query("SELECT m FROM Member m JOIN FETCH m.team")
```

단, `JpaRepository`에서 제공하는 것은 아니고 JPQL로 직접 작성해야 한다.

**Fetch Join 단점**

1. 쿼리 한 번에 모든 데이터를 가져오기 때문에 페이징 기법 사용 불가능
2. 1:N 관계가 두 개 이상인 경우 사용 불가능
3. 패치 조인 대상에게 별칭(`as`) 사용 불가능
4. 번거롭게 쿼리문을 작성해야 함 (JPQL)

### 2. DTO 직접 조회

엔티티가 아니라 필요한 필드만 뽑아오기.

```
@Query("SELECT new com.x.MemberDto(m.name, t.name) FROM Member m JOIN m.team t")
```

### 3. @BatchSize

N개를 IN 절로 묶어 조회 (1 + N → 1 + N/배치사이즈). 컬렉션 페이징과 함께 쓸 때 유용하다.

```
@BatchSize(size = 100)
@OneToMany(mappedBy = "team")
private List<Member> members;
```

### 4. @EntityGraph

Spring Data JPA의 어노테이션 방식. JPQL 안 쓰고 깔끔하게 처리.

```
@EntityGraph(attributePaths = {"team"})
List<Member> findAll();
```

---

# Q3. 지연 로딩과 즉시 로딩의 차이는?

## 상황 가정

`Player` 테이블(M)과 `Team` 테이블(1)이 있고 일대다 관계인 상황이다.

```
@Entity
public class Member {
    @ManyToOne(fetch = FetchType.EAGER)  // 즉시 로딩
    @ManyToOne(fetch = FetchType.LAZY)   // 지연 로딩
    private Team team;
}

Player player = em.find(Player.class, 1L); // 플레이어 조회 진행
```

## 차이점

- **즉시 로딩**: `em.find()`로 Player를 조회했을 때 연관관계에 있는 Team 테이블까지 한 번에 조회한다.
- **지연 로딩**: `em.find()`로 Player를 조회했을 때 Player만 조회하고, 연관관계 테이블 조회는 미룬다.
    - 지연 로딩을 할 경우 `team` 필드는 **프록시 객체**가 되고, 실제 SQL이 나가진 않는다.

## 어떤 것을 사용해야 하는가?

- 비즈니스 로직 상 Player가 필요한 곳 대부분에 Team 데이터 또한 필요한 경우 `FetchType.EAGER`을 통해 Team 테이블까지 조회하는 것이 좋을 것이다.
- 반면, 굳이 Team 데이터가 필요하지 않다면 `FetchType.LAZY`로 설정하여 Player만 조회하고, Team 데이터가 필요할 때 쿼리를 한 번 더 날리는 방식이 좋을 것이다.

---

# Q4. JPQL이란?

**JPQL**(Java Persistence Query Language) — JPA 표준 쿼리 언어

- DB 테이블이 아니라 **엔티티(객체)를 대상**으로 하는 객체지향 쿼리 언어이다.
- SQL과 문법이 매우 유사하지만 엔티티명으로 작성한다는 차이점이 있다.
- **동작 과정**: JPA가 JPQL을 분석해서 적절한 SQL로 변환 후 실행

## 등장 이유

JPA만으로는 100% 모든 쿼리를 커버할 수 없기 때문이다.

만약 여러 행을 반환하는 상황이거나 검색 조건을 넣어서 조회를 할 때는 어떻게 할 것인가? 이런 경우 `em.find(Member.class, 1L);` 만으로는 부족하다. 왜냐하면 실제로 데이터를 조회할 때는 "나이 20세 이상" 또는 "이름이 Yoo로 시작하는 회원" 등의 조건이 붙기 때문이다.

즉, DB에서 조건 없이 모든 데이터를 다 가져와 객체로 만든 뒤 자바에서 필터링하는 건 사실상 불가능하다.

> **WHERE 조건절이 필요하다 ⇒ 쿼리 언어가 필요하다 ⇒ 객체 중심 쿼리 언어 JPQL 필요함**

```
-- SQL — 테이블·컬럼 대상 (DB 테이블 중심)
SELECT * FROM MEMBER WHERE name = 'kim'      -- MEMBER = 테이블, name = 컬럼

-- JPQL — 엔티티·필드 대상 (객체 중심)
SELECT m FROM Member m WHERE m.name = 'kim'  -- Member = 클래스, m.name = 필드
```

## JPQL 특징

1. `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `JOIN` 등 쿼리 명령문 사용 가능
2. SQL을 추상화한 객체 지향적 쿼리 언어
3. 특정 DB에 의존적, 종속적이지 않다.

## JPQL 단점

1. 컴파일 시점에 검증 불가능 (QueryDSL로 극복)
2. 동적 쿼리 작성의 어려움 (QueryDSL로 극복)
3. 복잡한 쿼리 표현력 한계
4. 페치 조인의 제약

---
# Q5. Fetch Join이란?

JPQL에서 연관된 엔티티를 **한 번의 쿼리로 함께 조회**하는 기능.
주로 N+1 문제를 해결할 때 많이 사용되는 최적화 기법으로, **SQL에는 없는 JPQL 전용 문법**이다.

```
List<Member> members = em.createQuery(
    "SELECT m FROM Member m JOIN FETCH m.team", Member.class
).getResultList();

for (Member m : members) {
    System.out.println(m.getTeam().getName());  // 추가 쿼리 없음
}
```

실행되는 SQL은 딱 한 번.

- `team` 필드가 프록시가 아닌 **실제 엔티티**로 채워진다.
- JOIN하면서 연관 엔티티까지 SELECT에 포함된다.
- 이후 `m.getTeam()` 호출해도 추가 SELECT 쿼리가 안 나간다.

## 일반 JOIN vs Fetch JOIN (아예 다름)

- **일반 JOIN**: SELECT절에 명시한 것만 가져오므로 `team` 필드는 프록시 객체이다.
- 이후 `m.getTeam()` 호출 시 추가 SELECT 쿼리 발생 → **N+1 문제**

## 주의할 점

`@OneToMany` 페치 조인의 경우, JPA는 'Team 객체 수 == Player 엔티티 수'로 판단한다.

- ex) Team A 3명, Team B 1명 ⇒ `team.size() = 4` ❌
- ⇒ **`DISTINCT` 키워드**를 사용하여 중복 엔티티 제거 ⇒ `team.size() = 2` ✅

---

# Q6. @EntityGraph란?

JPA에서 엔티티와 그 연관 관계를 로드(fetch)할 때, **특정 연관 필드에 대한 fetch 전략을 동적으로 정의**할 수 있는 기능이다.

`@EntityGraph`가 적용된 메서드는 JPA가 엔티티 그래프를 참고하여 SQL 쿼리를 생성한다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    
    @EntityGraph(attributePaths = {"team", "orders"})
    Optional<Member> findById(Long id);
}
// => team, orders라는 엔티티를 함께 fetch한다.
```

## 핵심

- **LEFT OUTER JOIN**으로 동작 (fetch join은 기본적으로 INNER JOIN)
- **N+1 문제 해결** 가능 (fetch join과 동일한 효과)
- JPQL을 작성하지 않아도 된다.
- 불필요한 연관 엔티티는 로딩하지 않도록 주의해야 한다.

---
# Q7. commit과 flush 차이점은?

## flush

- 영속성 컨텍스트 변경사항 → DB로 SQL 전송하여 **동기화** 진행
- DB와 영속성 컨텍스트의 스냅샷을 일치시키는 작업
- 아직 **트랜잭션 내부**임 (커밋보다 앞선 시점)
- 아직 **롤백 가능**한 단계

## commit

- **트랜잭션 종료** / 모든 변경 사항 확정함
- 내부적으로 `flush()`를 실행한다.
- **DB 영구 반영**
- **롤백 불가능**