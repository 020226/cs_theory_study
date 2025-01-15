# 스프링부트 기초 수업 정리
[강의링크](https://www.youtube.com/watch?v=MDf6Q8TDHoo)
[수업페이지](https://www.slog.gg/p/13890)
[점프 투 스프링부트](https://wikidocs.net/book/7601)

### 목차
[0. 세팅](#0-세팅)
[1.JPA, ORM 개념 설명](#1-jpa-orm-개념-설명)

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
    prefix: file:src/main/resources/templates/ # 타임리프 캐시 끄기(이 설정을 해야 꺼짐)
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
  private List<Answer> answerList = new ArrayList<>();

  // 외부에서 answerList 필드에 접근하는 것을 차단
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
src/test/../QnaServiceApplicationTests.jav
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
    q2.addAnswer(a1); // 질문과 답변을 한 로직을 통해서 처리    a1.setCreateDate(LocalDateTime.now());
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
    questionRepository.save(q);
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
  // 테스트 코드에서는 Transactional을 붙여줘야 한다.
  // findById 메서드를 실행하고 나면 DB가 끝어지기 때문에
  // Transactional 어노테이션을 사용하면 메서드가 종료될 때까지 DB연결이 유지된다.
  @Transactional
  @Test
  @DisplayName("질문을 통해 답변 찾기")
  void t011() {
    // SQL : SELECT * FROM question WHERE id = 2;
    Optional<Question> oq = questionRepository.findById(2);
    assertTrue(oq.isPresent());
    Question q = oq.get();
    List<Answer> answerList = q.getAnswerList();
    assertEquals(1, answerList.size());
    assertEquals("네 자동으로 생성됩니다.", answerList.get(0).getContent());
  }
}
```
@Autowired 애너테이션
- questionRepository 변수는 선언만 되어 있고 그 값이 비어 있다. 
- 하지만 @Autowired 애너테이션을 해당 변수에 적용하면 스프링 부트가 questionRepository 객체를 자동으로 만들어 주입한다.
- 테스트 코드 작성 시에만 @Autowired를 사용하고 실제 코드 작성 시에는 생성자를 통한 객체 주입 방식을 사용


강의 21강
교재 2-05
까지 완료































