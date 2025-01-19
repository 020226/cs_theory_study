# 스프링부트 기초 수업 정리
[강의링크](https://www.youtube.com/watch?v=MDf6Q8TDHoo)\
[수업페이지](https://www.slog.gg/p/13890)\
[점프 투 스프링부트](https://wikidocs.net/book/7601)

### 목차
[0. 세팅](#0-세팅)\
[1.JPA, ORM 개념 설명](#1-jpa-orm-개념-설명)\

---
### 0. 세팅
/build.gradle
```
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	compileOnly 'org.projectlombok:lombok'
	developmentOnly 'org.springframework.boot:spring-boot-devtools'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testRuntimeOnly 'org.junit.platform:junit-platform-launcher'

    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	runtimeOnly 'org.mariadb.jdbc:mariadb-java-client'

    testImplementation 'org.junit.jupiter:junit-jupiter'
	testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}
tasks.named('test') {
	useJUnitPlatform()
	jvmArgs '-Xshare:off' // JVM 아규먼트 설정
}
```

db - mariaDB
db/db.sql
```sql
# DB 삭제, 생성, 선택
DROP DATABASE IF EXISTS qna_service;
CREATE DATABASE qna_service;
USE qna_service;

# 외래키 제약 비활성화
SET FOREIGN_KEY_CHECKS = 0;
# TRUNCATE 로 초기화한 데이터는 데이터 추가시 1번 번호로 시작한다.
TRUNCATE answer;
TRUNCATE question;
# 외래키 제약 활성화
SET FOREIGN_KEY_CHECKS = 1;
```

src/main/resources/application.yml
```yml
spring:
  profiles:
    active: dev # 기본적으로 dev 환경임을 명시
  output:
    ansi:
      enabled: always # 콘솔 색상 변경
  thymeleaf:
    cache: false # 타임리프 캐시 끄기
    prefix: classpath:/templates/ # file:src/main/resources/templates/ <- 이유는 모르겠으나 오류
    devtools:
      livereload:
        enabled: true
      restart:
        enabled: true
  datasource:
    url: jdbc:mariadb://127.0.0.1:3306/qna_service?useUnicode=true&characterEncoding=utf8&autoReconnect=true&serverTimezone=Asia/Seoul
    username: root
    password: 
    driver-class-name: org.mariadb.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: update # 옵션: none, validate, update, create, create-drop
    properties:
      hibernate:
        show_sql: true # 실행되는 SQL 쿼리 확인
        format_sql: true # 출력되는 SQL을 포맷팅
        use_sql_comments: true
```

홈 컨트롤러 도입
src/main/java/com/sbs/qna_service/boundedContext/home/HomeController.java
```java
@Controller
public class HomeController {
  @GetMapping("/home/main")
  @ResponseBody
  public String showHome() {
    return "안녕";
  }
}
```


### 1. JPA, ORM 개념 설명
String Data JPA -> 하이버네이트(JPA 구현체) -> JDBC -> MariaDB
- JPA라는 기술을 이용해서 DB와 소통할 수 있도록 해주는 도구

String Data JPA -> 추상화된 기술
- 추상화 -> 사람이 이해하기쉽다!!

findById
- SELECT * `member` WHERE id = 1;

ORM이란 -> Object-Relational Mapping
- 객체와 DBMS와 매핑해주는 기술
- 데이터 저장을 편리하게 해주는 도구!


### 2. 질문, 답변 엔티티 생성

엔티티는 데이터베이스 테이블과 매핑되는 자바 클래스를 말한다. 
우리가 만들고 있는 SBB는 질문과 답변을 할 수 있는 게시판 서비스이므로 SBB의 질문과 답변 데이터를 
저장할 데이터베이스 테이블과 매핑되는 질문과 답변 엔티티가 있어야 한다.

/boundedContext/home/answer/Answer.java
```java
@AllArgsConstructor
@NoArgsConstructor
@Data
@Entity
public class Answer {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Integer id;
  @Column(columnDefinition = "TEXT")
  private String content;
  private LocalDateTime createDate;
  @ManyToOne
  private Question question; // question_id 생성
}
```

게시판 서비스에서는 하나의 질문에 답변은 여러 개가 달릴 수 있다. 
따라서 답변은 Many(많은 것)가 되고 질문은 One(하나)이 된다. 
즉, @ManyToOne 애너테이션을 사용하면 N:1 관계를 나타낼 수 있다
@ManyToOne은 부모 자식 관계를 갖는 구조에서 사용한다. 
여기서 부모는 Question, 자식은 Answer라고 할 수 있다.

/boundedContext/home/question/Question.java
```java
@AllArgsConstructor
@NoArgsConstructor
@Data
@Entity // 스프링부트가 Question를 Entity로 본다.
public class Question {
  @Id // PRIMARY KEY
  @GeneratedValue(strategy = GenerationType.IDENTITY) // AUTO_INCREMENT
  private Integer id; // INT id
  @Column(length = 200) // VARCHAR(200)
  private String subject;
  @Column(columnDefinition = "TEXT") // TEXT
  private String content;
  private LocalDateTime createDate; // DATETIME
  // CascadeType.REMOVE : 질문이 삭제되면 답변도 같이 삭제된다.
  @OneToMany(mappedBy = "question", cascade = CascadeType.REMOVE)
  // @Transactional을 사용하고 싶지 않다면 @OneToMany(mappedBy = "question", cascade = CascadeType.REMOVE, fetch = FetchType.EAGER)
  private List<Answer> answerList = new ArrayList<>();

  // 외부에서 answerList 필드에 접근하는 것을 차단 = 캡슐화
  public void addAnswer(Answer a) {
    a.setQuestion(this); // Question 객체에 Answer 추가
    answerList.add(a); // Answer 객체에 Question 설정
  }
}
```

답변과 질문이 N:1 관계라면 질문과 답변은 1:N 관계라고 할 수 있다. 
이런 경우에는 @ManyToOne이 아닌 @OneToMany 애너테이션을 사용한다. 
질문 하나에 답변은 여러 개이므로 Question 엔티티에 추가할 Answer 속성은 List 형태로 구성해야 한다.

@Id 애너테이션
- id 속성에 적용한 @Id 애너테이션은 id 속성을 기본키로 지정한다.

@GeneratedValue 애너테이션
- 데이터를 저장할 때 해당 속성에 값을 일일이 입력하지 않아도 자동으로 1씩 증가하여 저장된다.

@Column 애너테이션
- 엔티티의 속성은 테이블의 열 이름과 일치하는데 열의 세부 설정을 위해 @Column 애너테이션을 사용한다.

### 3. 엔티티 종류
모든 엔티티는 관계가 있다.

1:1 : 한 엔티티가 다른 엔티티와 하나의 관계를 맺는다.
- 회원은 하나의 프로필은 갖는다.
- 프로필은 1명의 사용자에게만 속한다.

1:N : 한 엔티티가 다른 여러 엔티티와 관계를 맺는다.
- 한 질문에 여러개의 답변을 달 수 있다.
- 회원 1명은 게시글을 여러 개 작성 가능하다.
- 한 게시물에 좋아요가 여러개 있을 수 있다.

M:N(Many-to-Many) : 여러 엔티티가 다수의 엔티티와 관계를 맺는다.
- 한 학생은 여러개의 수업을 들을 수 있다.
- 하나의 수업에는 여러 학생이 참여할 수 있다.

### 4. 리포지터리
1. 질문 데이터 저장
2. 질문 데이터 조회
3. 질문 데이터 수정
4. 질문 데이터 삭제
5. 답변 데이터 생성 후 저장
6. 답변 조회
7. 답변에 연결된 질문 찾기/질문에 달린 답변 찾기
8. @Transactional
9. fetch=FetchType.EAGER
10. @Transactional = 자동 롤백, @Rollback(false)
11. 정리

#### 데이터 조회 관련 메서드 조합
|항목|예제|설명|
|-----|---|---|
|And|findBySubjectAndContent(String subject, String content)|여러 컬럼을 and 로 검색|
|Or|findBySubjectOrContent(String subject, String content)|여러 컬럼을 or 로 검색|
|Between|findByCreateDateBetween(LocalDateTime fromDate, LocalDateTime toDate)|컬럼을 between으로 검색|
|LessThan|findByIdLessThan(Integer id)|작은 항목 검색|
|GreaterThanEqual|findByIdGraterThanEqual(Integer id)|크거나 같은 항목 검색|
|Like|findBySubjectLike(String subject)|like 검색|
|In|findBySubjectIn(String[] subjects)|여러 값중에 하나인 항목 검색|
|OrderBy|findBySubjectOrderByCreateDateAsc(String subject)|검색 결과를 정렬하여 전달|

#### And(여러 칼럼을 and로 검색): findBySubjectAndContent
```sql
SELECT *
FROM question
WHERE subject = ?
AND content = ?
```
#### Or(여러 컬럼을 or로 검색): findBySubjectOrContent
```sql
SELECT *
FROM question
WHERE subject = ?
OR content = ?
```
#### Between(컬럼을 between으로 검색): findByCreateDateBetween
```sql
SELECT *
FROM question
WHERE create_date BETWEEN ? AND ?
```
#### LessThan(작은 항목 검색): findByIdLessThan
```sql
SELECT *
FROM question
WHERE id < ?
```
#### GreaterThanEqual(크거나 같은 항목 검색): findByIdGraterThanEqual
```sql
SELECT *
FROM question
WHERE id >= ?
```
#### Like(like 검색): findBySubjectLike
```sql
SELECT *
FROM question
WHERE subject LIKE ?
```
#### In(여러 값중에 하나인 항목 검색): findBySubjectIn
```sql
-- IN을 사용하면 이렇게 편리하게 작성할 수 있습니다.
SELECT *
FROM question
WHERE subject IN (?, ?);

-- 참고로 IN 없이 아래처럼 작성해야 합니다. IN 최고!
SELECT *
FROM question
WHERE subject = ? 
OR subject = ?;
```
#### OrderBy(검색 결과를 정렬하여 전달): findBySubjectOrderByCreateDateAsc
```sql
SELECT *
FROM question
WHERE subject = ?
ORDER BY create_date ASC
```

---

생성한 QuestionRepository 인터페이스를 리포지터리로 만들기 위해 JpaRepository 인터페이스를 상속한다.
JpaRepository는 JPA가 제공하는 인터페이스 중 하나로 CRUD 작업을 처리하는 메서드들을 이미 내장하고 있어 데이터 관리 작업을 좀 더 편리하게 처리할 수 있다.
- interface로 만들면 @Repository가 생략되어 있음
- JpaRepository<Question, Integer> -> JpaRepository<엔티티 클래스 이름, id 타입>
  /boundedContext/question/QuestionRepository.java
```java
public interface QuestionRepository extends JpaRepository<Question, Integer> {
  Question findBySubject(String subject);
  Question findBySubjectAndContent(String subject, String content);
  List<Question> findBySubjectLike(String subject);
  // 테스트 케이스 독립성 확보
  @Modifying // INSERT, UPDATE, DELETE와 같은 데이터가 변경 작업에서만 사용
  // nativeQuery = true 여야 MySQL 쿼리 사용이 가능하다.
  @Transactional
  @Query(value = "ALTER TABLE question AUTO_INCREMENT = 1", nativeQuery = true)
  void clearAutoIncrement();
}
```
/boundedContext/answer/AnswerRepository.java
```java
public interface AnswerRepository extends JpaRepository<Answer, Integer> {
  @Modifying // INSERT, UPDATE, DELETE와 같은 데이터가 변경 작업에서만 사용
  // nativeQuery = true 여야 MySQL 쿼리 사용이 가능하다.
  @Transactional
  @Query(value = "ALTER TABLE answer AUTO_INCREMENT = 1", nativeQuery = true)
  void clearAutoIncrement();
}
```

findBy + 엔티티의 속성명(예를 들어 findBySubject)과 같은 리포지터리의 메서드를 작성하면 입력한 속성의 값으로 데이터를 조회 가능

src/test/../QnaServiceApplicationTests.java
```java
@SpringBootTest // 스프링 부트의 테스트 클래스임을 의미
class QnaServiceApplicationTests {
  @Autowired // 필드 주입
  private QuestionRepository questionRepository;
  @Autowired
  private AnswerRepository answerRepository;

  @BeforeEach // 테스트 케이스 실행 전에 딱 한번 실행
  void beforeEach() {
    // 모든 데이터 삭제
    questionRepository.deleteAll();
    // 흔적삭제(다음번 INSERT 때 id가 1번으로 설정되도록)
    questionRepository.clearAutoIncrement();
    answerRepository.deleteAll();
    answerRepository.clearAutoIncrement();

    Question q1 = new Question();
    q1.setSubject("sbb가 무엇인가요?");
    q1.setContent("sbb에 대해서 알고 싶습니다.");
    q1.setCreateDate(LocalDateTime.now());
    questionRepository.save(q1);  // 첫번째 질문 저장

    Question q2 = new Question();
    q2.setSubject("스프링부트 모델 질문입니다.");
    q2.setContent("id는 자동으로 생성되나요?");
    q2.setCreateDate(LocalDateTime.now());
    questionRepository.save(q2);  // 두번째 질문 저장

    Answer a1 = new Answer();
    a1.setContent("네 자동으로 생성됩니다.");
    q2.addAnswer(a1); // 질문과 답변을 한 로직을 통해서 처리    
    a1.setCreateDate(LocalDateTime.now());
    answerRepository.save(a1);
  }
  @Test
  // @DisplayName : 테스트의 의도를 사람이 읽기 쉬운 형태로 설명
  @DisplayName("데이터 저장하기")
  void t001() {
    Question q = new Question();
    q.setSubject("겨울 제철 음식으로는 무엇을 먹어야 하나요?");
    q.setContent("겨울 제철 음식을 알려주세요.");
    q.setCreateDate(LocalDateTime.now());
    questionRepository.save(q); // 세번째 질문 저장
    assertEquals("겨울 제철 음식으로는 무엇을 먹어야 하나요?", questionRepository.findById(3).get().getSubject());
  }
    /*
    SQL
    SELECT * FROM question;
    */
  @Test
  @DisplayName("findAll")
  void t002() {
    List<Question> all = questionRepository.findAll();
    // 테스트에서 예상한 결과와 실제 결과가 동일한지를 확인하는 목적
    // assertEquals(기댓값, 실젯값) -> 기댓값과 실젯값이 동일하지 않다면 테스트는 실패
    assertEquals(2, all.size()); //  데이터 사이즈가 2인지 확인
    Question q = all.get(0);
    assertEquals("sbb가 무엇인가요?", q.getSubject());
  }
  /*
  SQL
  SELECT *
  FROM question
  WHERE id = 1;
  */
  @Test
  @DisplayName("findById")
  void t003() {
    // 리턴 타입 Optional인 이유
    // findById로 호출한 값이 존재할 수도 있고, 존재하지 않을 수도 있기 때문
    Optional<Question> oq = questionRepository.findById(1);
    if(oq.isPresent()) { // Optional -> isPresent : 값의 존재를 확인
      Question q = oq.get(); // get() 메서드를 통해 실제 Question 객체의 값 가져옴
      assertEquals("sbb가 무엇인가요?", q.getSubject());
    }
  }
  /*
  SQL
  SELECT *
  FROM question
  WHERE subject = 'sbb가 무엇인가요?';
  */
  @Test
  @DisplayName("findBySubject")
  void t004() {
    Question q = questionRepository.findBySubject("sbb가 무엇인가요?");
    assertEquals(1, q.getId());
  }
  /*
  SQL
  SELECT *
  FROM question
  WHERE subject = 'sbb가 무엇인가요?'
  AND content = 'sbb에 대해서 알고 싶습니다.';
  */
  @Test
  @DisplayName("findBySubjectAndContent")
  void t005() {
    Question q = questionRepository.findBySubjectAndContent(
        "sbb가 무엇인가요?", "sbb에 대해서 알고 싶습니다.");
    assertEquals(1, q.getId());
  }
  /*
  SQL
  SELECT *
  FROM question
  WHERE subject LIKE 'sbb%';
  */
  @Test
  @DisplayName("findBySubjectLike")
  void t006() {
    List<Question> qList = questionRepository.findBySubjectLike("sbb%");
    Question q = qList.get(0);
    assertEquals("sbb가 무엇인가요?", q.getSubject());
  }
  /*
	UPDATE question
	SET content = ?,
	create_date = ?,
	subject = ?
	WHERE id = ?
	*/
  @Test
  @DisplayName("데이터 수정하기")
  void t007() {
    // SELECT * FROM question WHERE id = 1;
    Optional<Question> oq = questionRepository.findById(1);
    assertTrue(oq.isPresent());
    Question q = oq.get();
    q.setSubject("수정된 제목");
    questionRepository.save(q); // update가 일어난 것
  }
  /*
	DELETE
	FROM question
	WHERE id = ?
	*/
  @Test
  @DisplayName("데이터 삭제하기")
  void t008() {
    // questionRepository.count()
    // SQL : SELECT COUNT(*) FROM question;
    assertEquals(2, questionRepository.count());
    Optional<Question> oq = questionRepository.findById(1);
    assertTrue(oq.isPresent());
    Question q = oq.get();
    questionRepository.delete(q);
    assertEquals(1, questionRepository.count());
  }
  /*
	특정 질문 가져오기
	SELECT *
	FROM question
	WHERE id = ?
	질문에 대한 답변 저장
	INSERT INTO answer
	SET create_date = NOW(),
	content = ?,
	question_id = ?;
	*/
  @Test
  @DisplayName("답변 데이터 생성 후 저장")
  void t009() {
    Optional<Question> oq = questionRepository.findById(2);
    assertTrue(oq.isPresent());
    Question q = oq.get();
		/*
		// v1
		Optional<Question> oq = questionRepository.findById(2);
		Question q = oq.get();
	 	*/
		/*
		// v2
		Question q = questionRepository.findById(2).orElse(null);
		*/
    Answer a = new Answer();
    a.setContent("네 자동으로 생성됩니다.");
    a.setQuestion(q);  // 어떤 질문의 답변인지 알기위해서 Question 객체가 필요하다.
    a.setCreateDate(LocalDateTime.now());
    answerRepository.save(a);
  }
    /*
	SELECT A.*, Q.*
	FROM answer AS A
	LEFT JOIN question AS Q
	on Q.id = A.question_id
	WHERE A.id = ?;
	*/
  @Test
  @DisplayName("답변 데이터 조회")
  void t010() {
    Optional<Answer> oa = answerRepository.findById(1);
    assertTrue(oa.isPresent());
    Answer a = oa.get();
    assertEquals(2, a.getQuestion().getId());
  }

  /*
    # EAGER를 사용한 경우
	SELECT Q.*, A.*
	FROM question AS Q
	LEFT JOIN answer AS A
	on Q.id = A.question_id
	WHERE Q.id = ?;
	*/
  // 테스트 코드에서는 Transactional을 붙여줘야 한다.
  // findById 메서드를 실행하고 나면 DB가 끝어지기 때문에
  // Transactional 어노테이션을 사용하면 메서드가 종료될 때까지 DB연결이 유지된다.
  // 테스트 + @Transactional = 자동 롤백, @Rollback(false)
  @Transactional // 메서드 내에서 트랜잭션이 유지된다! 실제 서버에서 JPA 프로그램들을 실행할 때는
  // DB 세션이 종료되지 않아 이와 같은 오류가 발생하지 않는다.
  // DB 세션이란 스프링 부트 애플리케이션과 데이터베이스 간의 연결을 뜻한다.
  @Test
  @DisplayName("질문을 통해 답변 찾기")
  @Rollback(false) // 테스트 메서드가 끝난 후에도 트랜젝션이 롤백되지 않고 커밋된다.
  void t011() {
    // SQL : SELECT * FROM question WHERE id = 2;
    Optional<Question> oq = questionRepository.findById(2);
    assertTrue(oq.isPresent());
    Question q = oq.get();
    // SQL : SELECT * FROM answer WHERE question_id = 2;
    List<Answer> answerList = q.getAnswerList();  // DB 통신이 끊긴 뒤 answer를 가져 옴 => 실패 - @Transactional 도입
    assertEquals(1, answerList.size());
    assertEquals("네 자동으로 생성됩니다.", answerList.get(0).getContent());
  }
}
```
@Autowired 애너테이션
- questionRepository 변수는 선언만 되어 있고 그 값이 비어 있다. 
- 하지만 @Autowired 애너테이션을 해당 변수에 적용하면 스프링 부트가 questionRepository 객체를 자동으로 만들어 주입한다.
- 테스트 코드 작성 시에만 @Autowired를 사용하고 실제 코드 작성 시에는 생성자를 통한 객체 주입 방식을 사용

**@Transactional `LAZY`와 `EAGER`**

@Transactional: 트랜잭션 시작, 커밋, 롤백이 자동으로 관리됨

커밋과 롤백(테스트 + @Transactional = 자동 롤백, @Rollback(false))
- @Transactional이 붙으면 롤백된다. 모든 데이터가 db에 저장되어 커밋되는 형태가 트랜젝션이고,
- 트랙젝션이 부튼 테스트 케이스의 경우 커밋이 되지 않는다 = 데이터가 저장되지 않는다
- 데이터를 저장시키고 싶다면? @Rollback(false)를 붙여주면 됨


이렇게 데이터를 필요한 시점에 가져오는 방식을 **지연(Lazy) 방식**이라고 한다. 
이와 반대로 q 객체를 조회할 때 미리 answer 리스트를 모두 가져오는 방식은 
**즉시(Eager) 방식**이라고 한다. @OneToMany, @ManyToOne 애너테이션의 옵션으로 
**fetch=FetchType.LAZY 또는 fetch=FetchType.EAGER**처럼 가져오는 방식을 설정할 수 있다.

```
@Transactional을 사용하고 싶지 않다면 Question 클래스에
@OneToMany(mappedBy = "question", cascade = CascadeType.REMOVE, fetch = FetchType.EAGER) 사용
```

- LAZY: 지연 로딩
  - @OneToMany, @ManyToOne -> fetch=FetchType.LAZY(질문이 답변에 접근하기 전까지 데이터베이스를 로딩시키지 않음)
- EAGER: 즉시 로딩
  - fetch=FetchType.EAGER를 통해 연관된 엔티티를 즉시 로딩. 
  - 즉시 db에서 연관된 데이터를 가져옴
- 트랜잭션: 데이터베이스 작업을 묶어서 처리하는 단위
  - 모든 작업이 성공한 경우 커밋하여 데이터 저장
    - 커밋 = 데이터 저장
  - 작업 중 문제가 발생하면 롤백해서 변경 사항을 취소
    - 롤백 = 변경사항을 저장 없이 취소 -> 데이터 일관성 유지 가능

LEFT JOIN vs. INNER JOIN
- INNER JOIN: 교집합(공통된 데이터를 포함하여 가져옴)
- LEFT JOIN: 공통된 데이터가 아닌 데이터도 누락시키지 않고 가져옴
- 예시) 질문 데이터는 1, 답변 데이터는 1, 2가 존재. 질문 1과 답변 2은 서로 참조 관계.
  - INNER JOIN: 질문1, 답변2만 가져옴. 답변1은 누락됨.
  - LEFT JOIN: 질문1, 답변1, 2 모두 가져옴

#### 리포지터리 총 정리
MVC 구조

Model
  - Service
  - Repository: DB와 소통하는 창구, SpringBoot JPA가 쿼리를 만들어줌
    - Repository의 JPA는 여러 가지 메서드를 가지고 있다.(findAll 등)
      - CrudRepository가 내장되어 있어 findById(), save(), findAll() 등을 사용할 수 있음
      - findBy~ 메서드들은 JPA가 내부적으로 사용할 수 있게 구현해뒀음(JPA의 룰이다)
      - 잘 모르겠으면 `Baeldung`에 검색이나 ai에 질문
View
Controller
  - db와 소통해서는 절대 안됨. repository를 통해야 됨.

Repository 과정:
SpringBoot JPA가 쿼리를 만들어서 DB에 넘기면 쿼리의 결과를 받아와 Controller 단에서 확인 후 View에 전달

데이터가 있는데 save를 하는 것 = update가 일어나는 것
findBy~: SELECT문

### 5. 템플릿 설정하기

**템플릿**은 자바 코드를 삽입할 수 있는 HTML 형식의 파일이다.
스프링부트는 템플릿 엔진을 지원하고 템플릿 엔진에는 Thymeleaf, Mustache, Groovy, Freemarker, Velocity 등이
있는데 스프링 진영에서는 타임리프 템플릿 엔진을 추천한다.

타임리프 설치하기(`build.gradle`파일 수정)
```
(... 생략 ...)

dependencies {
    (... 생략 ...)
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect'
}

(... 생략 ...)
```

http://localhost:8080/ 접속하면 /../src/main/resources/static/index.html 가 루트 경로의 페이지로 보여지게 된다.
static: 정적 리소스. 

템플릿 규칙: /../src/main/resources/templates 안에 작성해주어야 한다!!


/../src/main/resources/templates/question_list.html
```html
<h1>질문 리스트</h1>
```

/../boundedContext/home/question/QuestionController
```java
@Controller
public class QuestionController {
  @GetMapping("/question/list")
    public String list() {
      return "question_list"; // question_list.html 템플릿 파일 이름 리턴
    }
}
```
**템플릿을 사용하면 @ResponseBody를 애너테이션은 사용해선 안 된다!**
@ResponseBody는 화면에 리턴값을 띄워달라는 의미이기 때문에 템플릿 파일을 보여주지 않으므로 사용 금지.
@ResponseBody를 사용할 시 문자열 question_list가 그대로 리턴됨


#### HTML 표 만들기

| id | subject | content |
|----|---------|---------|
| 1  | sbb가 무엇인가요?| sbb에 대해서 알고 싶습니다.|
| 2  | 스프링부트 모델 질문입니다.	| id는 자동으로 생성되나요?|
```html
<table border="1">
	<colgroup>
		<col width="60px">
		<col width="250px">
		<col width="250px">
	</colgroup>
	<thead>
		<tr>
			<th>id</th>
			<th>subject</th>
			<th>content</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>1</td>
			<td>sbb가 무엇인가요?</td>
			<td>sbb에 대해서 알고 싶습니다.</td>
		</tr>
		<tr>
			<td>2</td>
			<td>스프링부트 모델 질문입니다.</td>
			<td>id는 자동으로 생성되나요?</td>
		</tr>
	</tbody>
</table>
```















