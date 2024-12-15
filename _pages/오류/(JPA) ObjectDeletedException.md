---
title: "(JPA) 삭제 상태 Entity(객체)를 merge()할 때 발생하는 ObjectDeletedException"
tags:
    - jpa
date: "2024-12-12"
thumbnail: "/assets/img/thumbnail/sample.png"
---
> *Spring Data JPA를 쓰고 있는 Repository 계층의 단위 테스트를 작성할 때 겪은 오류이다.*

## 1. 오류 발생 상황
### 전체 코드
```java
@DataJpaTest
public class ResponseExampleRepositoryTest {

    @Autowired
    ResponseExampleRepository responseExampleRepository;

    private List<ResponseExample> mockResponseExamples;

    @BeforeEach
    void setUp() {
        mockResponseExamples = List.of(  
            createExample("피드뷰 - 예시 프롬프트1", PictureRatio.RATIO_SERO, null, FALSE), 
            createExample("피드뷰 - 예시 프롬프트2", PictureRatio.RATIO_SERO, null, FALSE),
            createExample("피드뷰 - 예시 프롬프트3", PictureRatio.RATIO_SERO, null, FALSE),
            // ..(중략)
        );
        responseExampleRepository.saveAll(mockResponseExamples);
    }

    @Test
    @DisplayName("피드뷰 - 조건에 맞는 예시가 없을 때 빈 리스트가 반환되는지 검증")
    void findAllFeedViewWhenNoMatchingData_Then_Return_emptyList() {
        //given
        responseExampleRepository.deleteAll();
        responseExampleRepository.saveAll(mockResponseExamples.subList(10, mockResponseExamples.size()));

        //when
        List<ResponseExample> result = responseExampleRepository.findAllByPromptOnlyIsFalse();

        //then
        assertThat(result).isEmpty();
    }
}
```

### 오류 메세지
- org.springframework.dao.InvalidDataAccessApiUsageException: org.hibernate.<span style="color:red">ObjectDeletedException</span>: deleted instance passed to merge

---
## 2. 오류 분석
### 코드 오류 부분
```java
@Test
@DisplayName("피드뷰 - 조건에 맞는 예시가 없을 때 빈 리스트가 반환되는지 검증")
void findAllFeedViewWhenNoMatchingData_Then_Return_emptyList() {
    responseExampleRepository.deleteAll();
    responseExampleRepository.saveAll(mockResponseExamples.subList(10, mockResponseExamples.size()));
    // ..(중략)
}
```

### 오류 분석 요약
- Hibernate의 <span style="color:red">merge()는 삭제 상태 객체를 처리할 수 없다.</span> 
    - merge()하려고 할 때 삭제된 객체가 포함되어 있다면 오류(ObjectDeletedException)를 던진다. 
<br>
- deleteAll()을 호출한 이후에 **삭제된 객체를 saveAll()로 다시 저장**하려고 해서 오류가 발생한 것이다. 
    - 테스트 메서드의 saveAll()에서 인자로 넣어준 객체들이 **setUp 메서드의 saveAll()에서 인자로 넣어준 객체들과 같은 객체**이기 때문이다.
    - saveAll()은 내부적으로 merge()를 사용한다.

### 오류 분석 상세
1. @DataJpaTest 어노테이션이 테스트 클래스에 붙어 있기 때문에, **각 테스트 메서드는 트랜잭션 내에서 실행**된다. 또한, 각 테스트 메서드는 테스트 클래스가 실행되는 동안 하나의 트랜잭션 범위 내에서 진행된다.

2. **@Transactional 어노테이션이 기본적으로 사용하는 전파 방식은 Propagation.REQUIRED이다.** REQUEST 옵션은 현재 트랜잭션이 없으면 새로 시작하고, 이미 트랜잭션이 있으면 해당 트랜잭션에 참여하는 동작을 한다. 
    - 테스트 메서드는 @Transcational을 사용하고 있으며, deleteAll()과 saveAll()를 호출하고 있다. 이때, deleteAll()과 saveAll()은 모두 내부적으로 @Transactional 어노테이션을 사용하고 있는데, 이 메서드들을 호출하는 테스트 메서드에서 트랜잭션이 존재하기 때문에 두 메서드 모두 새로운 트랜잭션을 시작하지 않고 이미 존재하는 트랜잭션(테스트 메서드 것)에 참여한다.
<br>
3. 스프링은 기본으로 트랜잭션 범위의 영속성 컨텍스트 전략을 사용한다. 이는 트랜잭션을 시작할 때 영속성 컨텍스트를 생성하고 트랜잭션이 끝날 때 영속성 컨텍스트를 종료한다는 의미이다. 또한, **트랜잭션이 같으면 같은 영속성 컨텍스트를 사용**한다.
    - 테스트 메서드 내의 deleteAll()과 saveAll()은 같은 트랜잭션(테스트 메서드 것)에 참여하고 있으므로 같은 영속성 컨텍스트를 사용한다. 
<br>
4. **<span style="color:red">결론 - 오류의 원인</span>**
    1. deleteAll()과 saveAll()이 같은 영속성 컨텍스트를 사용하고 있다.
    2. 1번의 이유만으로는 오류가 발생하진 않는다. 그러나 나는 1번의 상황 속에서 deleteAll()에서 삭제된 객체를 saveAll()에서 사용하기 때문에 오류가 발생한 것이다.

---
## 3. 오류 해결
### 방법 1 : 삭제된 객체를 다시 저장하지 않고, 새로운 객체를 생성하여 저장
```java
void findAllFeedViewWhenNoMatchingData_Then_Return_emptyList() {
    // given
    responseExampleRepository.deleteAll();
    List<ResponseExample> newObjects = mockResponseExamples.subList(10, mockResponseExamples.size())
            .stream()
            .map(example ->
                    createExample(example.getExamplePrompt(),
                                example.getPictureRatio(),
                                example.getType(),
                                example.getPromptOnly())) // 새 객체 생성
            .collect(Collectors.toList());
    responseExampleRepository.saveAll(newObjects);
    // ...(동일)
}
```

### 방법 2 : 영속성 컨텍스트 초기화
```java
@PersistenceContext
private EntityManager entityManager;

void findAllFeedViewWhenNoMatchingData_Then_Return_emptyList() {
    // given
    responseExampleRepository.deleteAll();
    entityManager.flush();  // DB와 동기화
                            // flush()로 데이터베이스에 반영된 변경 사항도 테스트가 끝난 후 롤백되므로 
                            // 최종적으로 데이터는 삭제되지 않은 상태로 유지된다!
    entityManager.clear();  // 영속성 컨텍스트 초기화
    responseExampleRepository.saveAll(mockResponseExamples.subList(10, mockResponseExamples.size()));
    // ...(동일)
}
```

---
- 참고 자료  
https://stackoverflow.com/questions/18358407/org-hibernate-objectdeletedexception-deleted-object-would-be-re-saved-by-cascad  
https://stackoverflow.com/questions/31335211/autowired-vs-persistencecontext-for-entitymanager-bean  
https://www.inflearn.com/community/questions/527211/%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8%EC%99%80-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-propagation?srsltid=AfmBOordQwcCFu6HJkovFm-_NBBIR6DOSjiUAWJYAvy0H_xmAGEZMWnG  
https://milenote.tistory.com/107  
https://insanelysimple.tistory.com/314  
ChatGPT