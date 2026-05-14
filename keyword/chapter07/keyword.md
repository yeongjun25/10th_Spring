# Q1. Page와 Slice

  ### **정의**

  Spring Data JPA에서 페이지네이션 결과를 담는 두가지 타입이다. 둘다 페이징 기능을 제공하고Pageable을 파라미터로 받지만, 반환값이 서로 다르다.

  **Paging**이란?

  대량의 데이터를 효율적으로 처리하고 사용자에게 필요한 양만큼씩 나누어 제공하는 기법이다.
    
  ---

  ### **왜 사용할까**

  전체 데이터 집합을 페이지로 분할하여 한 번에 적은 양의 데이터만 가져온다.

  ⇒ 메모리 사용 줄임, 네트워크 트래픽 최소화, 더 빠른 응답 시간 제공
    
  ---

  #### **1. Page**

  전체 페이지 수, 현재 페이지 번호, 페이지 당 항목 수, 총 항목 수, 현재 페이지에 포함된 데이터 목록 등 페이징 관련 정보를 포함한다.

    - 페이지 번호를 1, 2, 3, 4. . . 명시하여 데이터 구분
    - 특정 페이지로 이동하거나 페이지 번호로 항목을 가져올 수 있다.
    - 전체 크기를 계산하는 쿼리가 필요하므로, 성능 비용이 발생할 수 있다.
    - 성능: 상대적으로 느림

    ---

  #### **2. Slice**

  전체 페이지 수나 총 항목 수에 대한 정보를 제공하지 않고, 현재 페이지에 포함된 데이터 목록과 다음 페이지가 있는지 여부에 대한 정보만 제공한다.

    - 전체 크기를 계산하지 않으므로 Page 객체보다 성능이 좋을 수 있다.
    - 전체 데이터 크기나 총 페이지 수를 굳이 알 필요가 없을 때 유용하다.
    - ‘무한 스크롤’ or ‘더보기 버튼’ 등의 방식처럼 페이지 정보가 없이 순차적으로 데이터를 로드하는 경우에 적합하다.
        - 성능: 상대적으로 빠름

    ---

  ### **예시 코드**

  `Pageable` 을 Repository 메서드의 파라미터에 넣는다.

    ```java
    // 가게 내 미션 목록 조회(페이징)
    public interface MissionRepository extends JpaRepository<Mission, Long> {
    
    	Page<Mission> findByStore_Id(Long storeId, Pageable pageable);
    	Slice<Mission> findSliceByStore_Id(Long storeId, Pageable pageable);
    }
    ```

  `PageRequest`를 Service 계층에서 만들고, 만들어진 결과값을 Repository에 넣는다.

    ```java
    public class MemberService {
    
    	 Pageable pageable = PageRequest.of(page, pageSize);
       Page<Mission> missionPage = missionRepository.findMissionsByRegion(
    	   member.getId(), 
    	   regionName, 
    	   pageable);
    }
    ```

  **Pageable** = 인터페이스

    ```java
    public interface Pageable {
        int getPageNumber();   // 몇 번째 페이지인지
        int getPageSize();     // 한 페이지에 몇개인지
        Sort getSort();        // 정렬 기준
        long getOffset();      // offset =  pageNumber * pageSize 계산
    }
    ```

  **PageRequest** = Pageable을 구현한 클래스 (실제 구현체)

    ```java
    흔히 쓰는 List - ArrayList 관계랑 똑같다.
    
    List<String> list = new ArrayList<>();
    Pageable     page = new PageRequest(...); 
    
    ```

  ### **핵심 한 줄**

    1. 전체 개수가 꼭 필요하고 페이지 정보가 필요하다 ⇒ Page
    2. 페이지 명시가 필요없고, 다음 페이지의 유무만 필요하다 ⇒ Slice
---
# Q2. Java stream API

  ### **정의**

  데이터 컬렉션(List, Set 등)을 함수형 스타일로 처리하기 위한 API이다. 데이터의 흐름을 ‘파이프라인’ 흐름처럼 처리한다고 생각할 수 있다.
    
  ---

  ### **예시 코드**

  Repository에서 조회한 엔티티 리스트를 응답 DTO로 변환하는 상황 (`map()`, `toList()` 주로 사용)

    ```java
    // 전통적인 for문 방식
    List<GetMission> result = new ArrayList<>();
    for (Mission mission : missionList) {
        if (mission.getReward() > 100) {
            result.add(MissionConverter.toGetMission(mission));
        }
    }
    
    // Stream 방식 
    List<GetMission> result = missionList.stream()
            .filter(m -> m.getReward() > 100)
            .map(MissionConverter::toGetMission)
            .toList();
    ```

  stream API를 통해 데이터를 추상화하고 다양한 연산을 수행할 수 있다. 또한, 간결한 표현으로 데이터를 다루므로 코드의 가독성과 유지보수성이 향상된다.
    
  ---

  #### Stream

  데이터 처리 연산을 지원하도록 소스에서 추출된 ‘연속된 요소’로, 연속된 값 집합의 인터페이스를 제공한다.

  #### **Stream의 3단계 구조**

    1. 소스: List, Page, Array, Set 등

                          ⬇️

    2. 중간연산 - filter(), map(), sorted() 등

                         ⬇️

    3. 최종연산 - toList(), collect(), count() 등

    ---

  #### Stream API 특징

    1. **재사용 불가능**: 스트림은 일회성이다. 한 번 사용된 스트림은 재사용 할 수 없고, 필요한 경우 다시 만들어서 사용해야 한다.
    2. **함수형**: 스트림 연산은 람다 표현식을 사용하여 함수형 프로그래밍을 하므로 내부 반복이 가능하다.
    3. **지연성 추구**: 스트림 연산들(filter, map)은 실제로 필요할 때까지 연산이 수행되지 않고 지연된다.
---
#Q3. 객체 그래프 탐색

  #### 정의

  JPA에서 연관관계로 매핑된 객체들을 마치 그래프 따라가듯 탐색하는 것

  #### **예시 코드**

    ```java
    School school = schoolRepository.findById(1L);
    
    // 객체 그래프 탐색
    String studentName = school.getStudents()    
                               .getName(); 
    ```

  JPA가 연관관계를 자동으로 따라가며 SQL JOIN으로 번역해주기 때문에 별도의 쿼리를 개발자가 직접 작성할 필요가 없다. 우리는 school 객체만 가지고 점(.) 하나로 연관 객체에 접근하면 된다.

  #### 주의사항

  점(.) 하나로 연관 객체에 접근하므로 편리하지만, 지연 로딩과 함께 사용할 때 N+1 쿼리 문제가 발생할 수 있다.

    ```java
    List<Student> students = studentRepository.findAll();   // 쿼리 1번
    
    // 각 학생의 학교를 지연로딩(LAZY)로 조회
     (Student s : students) {
        s.getSchool().getName();   // 멤버 1명마다 쿼리 1번씩 -> 총 N+1번 쿼리 발생
    }
    ```

---
#Q4. @Valid vs @Validated

  둘다 검증할 때 사용하는 어노테이션이지만 적용범위, 기능이 다르다.

  #### @Valid

  자바 표준 스펙 기술로, 객체나 메서드 파라미터의 유효성 검사할 때 주로 사용된다.

  보통 **DTO** 속 필드에 `@Email`, `@NotNull`, `@Positive` 등의 어노테이션을 붙여 데이터 값의 허용 범위를 정하고, **Controller**에서 `@Valid`로 적절한 입력값이 들어갔는지 검증한다.

  #### **예시 코드**

    ```java
    public class MemberRequestDto{
        @Email
        private String email;
    
        @NotNull
        private Long id;
    }
    ```

    - email 필드에 이메일 형태가 아닌 값이 들어가는 것을 방지한다.
    - id 필드에 Null이 들어가는 것을 방지한다.

    ```java
    // 회원가입
        @PostMapping("/signup")
        public ApiResponse<MemberResponseDTO.SignUpDTO> signUp(
                @RequestBody @Valid MemberRequestDTO.SignUpDTO requestDto
        ) {
            return ApiResponse.onSuccess(MemberSuccessCode.CREATED, null);
        }
    ```

    - Controller에서 @Valid 검증 문제 발생 ⇒ **MethodArgumentNotValidException 예외 처리**

    ---

  #### @Validated

  @Valid가 자바 표준 기술인 반면, @Validated는 스프링에서 제공하는 기술이다.

  @Valid는 주로 Controller 계층에서 사용되고, @Validated는 다른 계층에서도 많이 사용된다.

  #### **예시 코드**

    ```java
    @RestController
    @Validated                         
    @RequiredArgsConstructor
    public class MemberController {
    
        @GetMapping("/members/{memberId}")
        public ApiResponse<?> getMyPage(
            @PathVariable @NotNull @Positive Long memberId  
        ) {
            return ApiResponse.onSuccess(memberService.getMyPage(memberId));
        }
     }
    ```
    
  ---

  #### 동작원리 차이

  **Valid**

  클라이언트 요청 → 디스패처 서블릿 → 적절한 Controller에게 요청 전달

  : 서블릿이 요청을 전달하는 과정에서 내부적으로 @Valid 어노테이션이 붙은 필드의 유효성을 검증한다.

  검증 실패 시 MethodArgumentNotValidException 예외 발생

  **Validated**

  AOP를 기반으로 메서드의 요청을 가로채 유효성 검증을 진행한다.

  검증 실패 시 ConstraintViolationException 예외 발생