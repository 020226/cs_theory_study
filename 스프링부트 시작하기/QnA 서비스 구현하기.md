# 스프링부트 기초 수업 정리 - 중지

## 주요 내용은 아래 `점프투스프링부트` 교재 참고하기!
## 이 페이지는 `정리한 주요 개념` 위주로 학습!
[강의링크](https://www.youtube.com/watch?v=MDf6Q8TDHoo)\
[수업페이지](https://www.slog.gg/p/13890)\
[점프 투 스프링부트](https://wikidocs.net/book/7601)

### 목차
[0. 세팅](#0-세팅)\
[1.JPA, ORM 개념 설명](#1-jpa-orm-개념-설명)\
[2. 질문 답변 엔티티 생성](#2-질문-답변-엔티티-생성)\
[3. 엔티티 종류](#3-엔티티-종류)\
[4. 리포지터리](#4-리포지터리)\
[5. 템플릿 설정하기](#5-템플릿-설정하기)\
[스프링부트에서-transactional은-커밋과-롤백을-담당한다](#34강-스프링부트에서-transactional은-커밋과-롤백을-담당한다)\
[fetch-전략-eager-lazy에-따른-sql-변화-설명](#35강-fetch-전략-eager-lazy에-따른-sql-변화-설명)\
[답변-등록-버튼-만들기](#39강-답변-등록-버튼-만들기)\
[질문을-저장하는-곳에서-처리하고-redirect-하는-방식](#41강-질문을-저장하는-곳에서-처리하고-redirect-하는-방식)\
[질문을-저장하는-곳에서-처리하고-redirect-하지-않고-본인이-직접-템플릿-실행하여-응답하는-방식](#42강-질문을-저장하는-곳에서-처리하고-redirect-하지-않고-본인이-직접-템플릿-실행하여-응답하는-방식)

[스프링 시큐리티](#스프링-시큐리티)
[회원가입](#회원가입)
[로그인](#로그인)
[ENUM](#enum)

---
### 기본 개념 정리
엔티티(Entity)의 개념:

- 엔티티는 데이터베이스의 테이블에 해당하는 객체로, 애플리케이션에서 다루는 데이터의 구조를 정의한다.

엔티티(Entity)의 특징:

- 각 엔티티는 고유 ID를 가지고 데이터베이스의 레코드를 객체 형태로 표현한다.

엔티티(Entity)의 사용 이유:

- 엔티티를 사용하면 데이터베이스와 상호작용을 객체 형태로 처리하여 가독성과 유지보수성에 이점이 있다.
- JPA를 통한 엔티티와 데이터베이스 간의 매핑을 자동 처리라는 이점이 있다.

리포지터리(Repository)의 개념:

- 데이터베이스와 상호작용을 추상화하는 인터페이스이다. CRUD 작업을 수행하는 메서드를 제공한다.

리포지터리(Repository)의 역할:

- 데이터 저장 및 조회 메서드를 제공하고 메서드 이름을 기반으로 자동으로 쿼리를 생성할 수 있다.

리포지터리(Repository)의 특징:

- JpaRepository 인터페이스를 상속받아 CRUD 메서드를 자동으로 사용한다.

리포지터리(Repository)의 사용 이유:

- 복잡한 SQL 쿼리를 작성하지 않아도 데이터에 접근할 수 있다는 이점이 있다.

엔티티와 리포지터리의 관계

- 엔티티와 리포지터리는 스프링부트에서 DB와의 상호작용을 관리하는 구성 요소이다.
  - 엔티티는 DB의 테이블 구조를 나타내고, 데이터 속성과 관계를 정의한다.
  - 리포지터리는 이 엔티티 객체의 CRUD 작업을 하고 엔티티에 대한 데이터 접근을 추상화하여 제공한다.

간단한 데이터베이스 연동 흐름

1. 엔티티 생성: 비즈니스 로직에서 필요한 데이터를 담기 위해 엔티티 객체를 생성한다.
2. 리포지터리 호출: 생성된 엔티티를 저장하거나 조회하기 위해 리포지터리 메서드를 호출한다.
3. 데이터베이스 작업: 리포지터리는 JPA를 통해 데이터베이스와 연결되어 요청된 작업을 수행한다.
4. 결과 반환: 데이터베이스에서 작업된 결과를 리포지터리가 반환하여 필요한 데이터를 사용할 수 있게 된다.

예제 코드 작성(게시판) + 엔티티 정의 + JpaRepository 인터페이스의 주요 기능 사용법

게시글을 나타내는 Post 엔티티 정의

```java
import javax.persistence.*;
@Entity // 해당 클래스가 엔티티임을 나타낸다.
@Data
public class Post {
    @Id // 엔티티의 고유 식별자 지정
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    // id 자동 생성
    private Long id;

    @Column(nullable = false) // 필드와 데이터베이스 컬럼 간의 맵핑
    private String title;

    @Column(nullable = false)
    private String content;
   }
```

Post 엔티티를 관리하는 PostRepository 정의

```java
// Ver1. 기본 리포지터리 정의
import org.springframework.data.jpa.repository.JpaRepository;
public interface PostRepository extends JpaRepository<Post, Long> {
}

// Ver2. 사용자 정의 메서드
// 메서드 이름을 기반으로 자동으로 쿼리를 생성하는 방식
// 특정 조건에 맞는 데이터를 검색할 수 있다
// 메서드 이름의 규칙을 따르면 JPA가 쿼리를 자동으로 생성
public interface PostRepository extends JpaRepository<Post, Long> {
    // 이름으로 조회
    PostfindByName(String name);
}
```

리포지터리를 통해 게시글을 저장, 조회하는 PostService 클래스 작성

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class PostService {
    @Autowired
    private PostRepository postRepository;
    // 게시글 저장 - 엔티티를 저장하거나 업데이트한다
    public Post savePost(Post post) { return postRepository.save(post); }
    // 모든 게시글 조회 - 주어진 ID로 엔티티를 조회한다
    public List<Post> getAllPosts() { return postRepository.findAll(); }
    // 게시글 삭제- 주어진 ID로 엔티티를 삭제한다
    public void deletePost(Long id) {
        postRepository.deleteById(id);
    }
}
```

애플리케이션 시작

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class DataLoader implements CommandLineRunner {
    private final PostService postService;

    public DataLoader(PostService postService) {
        this.postService = postService;
    }

    @Override
    public void run(String... args) {
        // 게시글 저장
        Post post1 = new Post();
        post1.setTitle("첫 번째 게시글");
        post1.setContent("안녕하세요.");
        postService.savePost(post1);

        Post post2 = new Post();
        post2.setTitle("두 번째 게시글");
        post2.setContent("반갑습니다.");
        postService.savePost(post2);

        // 모든 게시글 조회
        System.out.println("모든 게시글:");
        postService.getAllPosts().forEach(post -> {
            System.out.println(post.getTitle() + " - " + post.getContent());
        });
    }
}
```

### MVC 구조

- MVC(Model-View-Controller) 패턴은 애플리케이션의 구조를 분리하여 코드의 가독성과 유지보수성을 노피는 디자인 패턴이다.
  - Model: 데이터와 비즈니스 로직을 담당한다. 데이터베이스와의 상호작용, 데이터 검증 및 비즈니스 규칙을 처리한다.
  - View: 사용자 인터페이스를 구성하며, 사용자에게 정보를 표시하고 사용자 입력을 받는다.
  - Controller: 사용자 요청을 처리하고, 적절한 Model과 View를 연결하여 결과를 반환한다.
- 필요 요소들
  - DTO (Data Transfer Object): DTO는 데이터 전송 객체로, 주로 데이터 전송을 목적으로 사용되는 객체이다. 여러 필드를 하나의 객체로 묶어 전달하기 때문에 데이터의 캡슐화를 제공한다. 또한 Controller ↔ Service, Service ↔ View 간의 데이터 전송을 용이하게 한다.
  - Service: 비즈니스 로직을 처리하는 계층으로, Controller와 Model 사이에서 데이터 접근을 추상화한다. 비즈니스 규칙을 구현하고, 데이터의 유효성을 검증하며, 트랜잭션 관리를 수행한다.
  - Repository: 데이터베이스와의 상호작용을 담당하는 계층으로, 데이터 CRUD 작업을 수행한다. JPA와 같은 ORM(Object-Relational Mapping) 프레임워크를 사용하여 데이터베이스에 접근할 수 있다.
  - Validation: 입력 데이터의 유효성을 검사한다. 스프링에서는 @Valid, @NotNull, @Size 등의 어노테이션을 통해 DTO의 필드에 대한 유효성 검사를 수행한다.
- MVC 구조 데이터 흐름
  1. 사용자 입력: 사용자가 View에서 데이터를 입력한다.
  2. 요청 처리: Controller가 요청을 받아 DTO로 변환한다.
  3. 비즈니스 로직 처리: Controller는 Service를 호출하여 비즈니스 로직을 수행한다.
  4. 데이터 저장: Service는 Repository를 통해 데이터베이스에 접근하여 데이터를 저장한다.
  5. 응답 반환: 처리 결과를 View에 전달하여 사용자에게 결과를 표시한다.
- Service가 필요한 이유:
  - 비즈니스 로직과 데이터 접근 로직을 분리하여 코드의 구조를 명확히 하고 유지보수를 쉽게 한다.
  - 즉, 새로운 기능 추가 시, 기존의 Controller와 Model을 변경하지 않고 Servcie 레이어에서만 변경할 수 있어 애플리케이션 확장성이 높아진다.

### 질문1 - @Transactional이란?

@Transactional은 스프링 프레임워크에서 제공하는 어노테이션으로, 메서드 또는 클래스에 트랜잭션을 적용할 수 있게 해준다. 트랜잭션은 데이터베이스에서 일어나는 작업의 단위를 정의하며, 여러 작업이 성공적으로 완료되거나 모두 롤백(취소)되어야 하는 경우에 사용된다.

- @Transactional을 사용하면 트랜잭션의 시작과 끝을 명시할 수 있다. 해당 메서드가 시작될 때 트랜잭션이 시작되고, 메서드 실행이 완료되면 트랜잭션이 커밋(저장)된다. 만약 예외가 발생하면 트랜잭션은 롤백된다.
- 데이터베이스의 일관성을 유지하기 위해 여러 작업을 하나의 트랜잭션으로 묶어 처리할 수 있다.

### 질문2 - fetch=FetchType.EAGER의 의미

fetch = FetchType.EAGER는 JPA(Java Persistence API)에서 엔티티의 연관 관계를 설정할 때 사용하는 옵션 중 하나이다. 연관된 엔티티를 어떻게 로드할지를 결정한다.

- FetchType.EAGER를 설정하면, 해당 엔티티를 조회할 때 연관된 엔티티도 함께 즉시 로드된다. 즉, 부모 엔티티를 조회할 때 자식 엔티티도 자동으로 조회하여 메모리에 로드하게 된다.
- EAGER 로딩을 설정한 경우, 부모 엔티티를 가져올 때 자식 엔티티가 항상 포함되므로, 사용자가 자식 엔티티에 접근할 때 추가적인 쿼리를 실행할 필요가 없다.

### 질문3 - LAZY란?

LAZY는 JPA(Java Persistence API)에서 엔티티의 연관 관계를 설정할 때 사용하는 옵션 중 하나로, 지연 로딩(Delayed Loading)을 의미한다. FetchType.LAZY를 설정하면, 연관된 엔티티가 실제로 필요할 때까지 로드하지 않도록 지연시키는 방식이다.

- FetchType.LAZY를 설정하면, 해당 엔티티를 조회할 때 연관된 엔티티는 즉시 로드되지 않고, 실제로 해당 연관된 데이터를 접근할 때 로드된다. 즉, 부모 엔티티를 조회하더라도 자식 엔티티는 처음에는 로드되지 않는다.
- 연관된 데이터에 접근할 때, JPA가 자동으로 추가 쿼리를 실행하여 데이터를 로드하여 필요한 시점에 로드할 수 있다.




### 0. 세팅
/build.gradle
```
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.4.1'
	id 'io.spring.dependency-management' version '1.1.6'
}

group = 'com.sbs'
version = '0.0.1-SNAPSHOT'

java {
	toolchain {
		languageVersion = JavaLanguageVersion.of(17)
	}
}

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

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

	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect'

	implementation 'org.springframework.boot:spring-boot-starter-validation'

	implementation 'org.springframework.boot:spring-boot-starter-security'
	implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity6'

	implementation 'org.commonmark:commonmark:0.21.0'
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
    username: sbsst
    password: sbs123414
    driver-class-name: org.mariadb.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: update # 옵션: none, validate, update, create, create-drop
    properties:
      hibernate:
        show_sql: true # 실행되는 SQL 쿼리 확인
        format_sql: true # 출력되는 SQL을 포맷팅
        use_sql_comments: true
  # LOGGING
  logging:
    level:
      org.hibernate.orm.jdbc.bind: TRACE
      org.hibernate.orm.jdbc.extract: TRACE
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
~30강까지

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


#### HTML 표 실습

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


/../src/main/resources/templates/question_list.html
```html
<h1>질문 리스트</h1>
[[${questionList}]]
<h2>[[${age}]]</h2>
<table border="1">
  <colgroup>
    <col width="60px">
    <col width="250px">
    <col width="250px">
  </colgroup>
  <thead>
  <tr>
    <th>번호</th>
    <th>제목</th>
    <th>작성날짜</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td>2</td>
    <td>스프링부트 모델 질문입니다.</td>
    <td>2024-12-21 10:08:58.783770</td>
  </tr>
  <tr>
    <td>1</td>
    <td>sbb가 무엇인가요?</td>
    <td>2024-12-21 10:08:58.767813</td>
  </tr>
  </tbody>
</table>
```

[[${questionList}]]: 타임리프 문법
이 문법을 사용하려면 @Data의 ToString에 의해 무한 순환됨.
현재 Question 클래스의 answerList와 Answer 클래스의 question 양방향 참조 관계이기 때문에
Question 클래스에서는 상관 없지만 Answer 클래스에서 toString이 작동하면 무한재귀 발생 => ***StackOverflowError***


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
  @ToString.Exclude // 무한순환참조 방지. ToString 대상에서 제외
  private Question question; // question_id 생성
}
```


/../boundedContext/home/question/QuestionController
```java
@Controller
@RequiredArgsConstructor // final이 붙은 것들 요청이 들어올 때만 생성해줌
public class QuestionController {
  private final QuestionRepository questionRepository; // Service 만들기 전이라 리포지터리가 컨트롤러에 있는 것


  @GetMapping("/question/list")
  public String list(Model model) { // model은 MVC의 모델
    List<Question> questionList = questionRepository.findAll(); // 리스트이기 때문에 전부를 가져옴 = finaAll
    model.addAttribute("questionList", questionList); // "questionList"이란 이름으로 questionList을 화면에 뿌려줌 
    return "question_list";
  }
}
```
**템플릿을 사용하면 @ResponseBody를 애너테이션은 사용해선 안 된다!**
@ResponseBody는 화면에 리턴값을 띄워달라는 의미이기 때문에 템플릿 파일을 보여주지 않으므로 사용 금지.
@ResponseBody를 사용할 시 문자열 question_list가 그대로 리턴됨


---

### 34강 스프링부트에서 @Transactional은 커밋과 롤백을 담당한다.

#### 트랜잭션 요약
```
트랜잭션 정의
여러 작업을 하나의 단위로 묶어 처리하는 것.

예시: 은행 이체

1번: A 계좌에서 돈을 출금.
2번: B 계좌에 돈을 입금.
두 작업이 모두 성공해야 완료.
1번 성공, 2번 실패 → 이전 상태로 롤백(되돌리기).
@Transactional 역할

작업 시작.
문제 없으면 커밋(완료).
오류 발생 시 롤백(되돌리기).
```

### 35강 Fetch 전략 EAGER, LAZY에 따른 SQL 변화 설명

```
Fetch 전략 
1. EAGER (즉시 로딩)
2. LAZY (지역 로딩)

LAZY (지역 로딩)
SELECT * FROM question WHERE id = 2;
SELECT * FROM answer WHERE question_id = 2;

EAGER (즉시 로딩)
SELECT Q.*, A.*
FROM question AS Q
LEFT JOIN answer AS A
ON Q.id = A.question_id 
WHERE Q.id = ? 
```

### 39강 답변 등록 버튼 만들기

#### 랜더링 결과
```
action="@{|/answer/create/${question.id}|}" -> 랜더링 안됨

th:action="@{/answer/create/${question.id}}"
 => action=/answer/create/${question.id}

th:action="@{|/answer/create/${question.id}|}"
 => action=/answer/create/1
```

### 41강 질문을 저장하는 곳에서 처리하고 redirect 하는 방식

#### 처리순서
```
현재 URL : /question/detail/1
폼의 textarea에 값을 "어서와"로 채우고 작성버튼 클릭
요청 : POST/answer/create/1
 - PAYLOAD: content: 어서와
 - 요청처리 : AnswerController::createAnswer 액션 메서드 실행
 - 요청처리 : Question question = questionService.getQuestion(id);
응답 : redirect:question/detail/1

요청: GET/question/detail/1
 - 요청처리 : QuestionController::detail 액션 메서드 실행
 - 요청처리 : Question question = questionService.getQuestion(id);
 - 요청처리 : model.addAttribute("question", question);
 - 요청처리 : question_detail 템플릿 실행
 - 응답 : 템플릿에 데이터가 결합된 최종 결과물(HTML)을 고객한테 전달.

최종 URL : question/detail/1
 - 요청을 날리기 전과 똑같은 화면이 보인다.
 ```

### 42강 질문을 저장하는 곳에서 처리하고 redirect 하지 않고 본인이 직접 템플릿 실행하여 응답하는 방식

```java
@PostMapping("/create/{id}")
public String createAnswer(Model model, @PathVariable("id") Integer id, @RequestParam(value="content") String content) {
    Question question = questionService.getQuestion(id);
    // TODO: 답변을 저장한다.
    
    model.addAttribute("question", question);
    
    // return "redirect:/question/detail/%s".formatted(id);
    return "question_detail"; // QuestionController::detail 메서드에서 사용하던 템플릿이다.
    // 이렇게 되면 고객이 다시 GET 요청할 필요가 없어진다. 위 방식과 비교해서 장단점이 있다.
}
```

#### 처리 순서
```
현재 URL : /question/detail/1
폼의 textarea에 값을 "어서와"로 채우고 작성버튼 클릭
요청 : POST/answer/create/1
 - PAYLOAD: content: 어서와
 - 요청처리 : AnswerController::createAnswer 액션 메서드 실행
 - 요청처리 : Question question = questionService.getQuestion(id);
 - 요청처리 : model.addAttribute("question", question);
 - 요청처리 : question_detail 템플릿 실행
 - 응답 : 템플릿에 데이터가 결합된 최종 결과물(HTML)을 고객한테 전달.
최종 URL : /answer/create/1
 - 요청을 날리기 전과 똑같은 화면이 보이지만 URL이 달라진다/
```


## 스프링 시큐리티

- 스프링 시큐리티(Spring Security) : 회원 가입과 로그인 기능에 필요함
- 스프링 기반 웹 애플리케이션의 인증과 권한을 담당하는 스프링의 하위 프레임워크
- 인증(authenticate) : 로그인과 같은 사용자의 신원을 확인하는 프로세스
- 권한(authorize) : 인증된 사용자가 어떤 일을 할 수 있는지(어던 접근 권한이 있는지) 관리하는 것

스프링시큐리티는 인증과 인가뿐 아니라 보안과 관련된 기능도 제공

인증과 인가
인증 : 신원 확인
- 사용자가 제공한 아이디와 비밀번호가 유효한지 확인
- 예 : 로그인 처리

인가 : 권한 부여
- 사용자가 특정 작업을 수행할 수 있는 권한 확인
- 예 : 게시물 작성자가 게시물 수정과 삭제에 관한 권한을 가짐

결론 : 스프링 시큐리티는 인증과 인가를 간단하게 구현할 수 있는 프레임워크이다.

`스프링 시큐리티 설치`
build.gradle 파일 수정

```
(... 생략 ...)

dependencies {
    (... 생략 ...)
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity6'
}

(... 생략 ...)
```

`스프링 시큐리티 설정하기`
SecurityConfig.java 파일 생성

```java
@Configuration // 스프링의 환경 설정 파일임을 의미
@EnableWebSecurity // 모든 요청 URL이 스프링 시큐리티의 제어를 받도록 만듦. 스프링 시큐리티 활성화.
public class SecurityConfig {
  // 내부적으로 SecurityFilterChain 클래스가 동작하여 모든 요청 URL에 이 클래스가 필터로 적용되어 URL 별로 특별한 설정을 하도록
  @Bean // SecurityFilterChain 빈을 생성
  SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    return http.build();
  }
}
```

### 빈
빈(bean)은 스프링에 의해 생성 또는 관리되는 객체를 의미한다. 우리가 지금껏 만들어 왔던 컨트롤러, 서비스, 리포지터리 등도 모두 빈에 해당한다. 또한 앞선 예처럼 @Bean 애너테이션을 통해 자바 코드 내에서 별도로 빈을 정의하고 등록할 수도 있다.

### 회원가입
회원 가입 기능을 구현하려면 회원 정보와 관련된 데이터를 저장하고 이를 관리하는 엔티티와 리포지터리 등을 만들어야 하고, 폼과 컨트롤러와 같은 요소를 생성해 사용자로부터 입력받은 데이터를 웹 프로그램에서 사용할 수 있도록 만들어야 한다.

`회원가입 실행 시 로직 순서`
1. userController::create 메서드 실행
2. 회원 가입 폼을 GET 방식으로 호출
3. 폼에서 데이터 입력이 제대로 이루어 지지 않은 경우
- 회원 가입 폼을 GET 방식으로 호출
- 입력이 안 된 경우
- 비밀번호가 틀린 경우
- 중복된 이름과 이메일로 회원가입 하는 경우
4. 폼에서 데이터가 제대로 입력되고 회원가입 버튼을 누른 경우
5. userController::create 메서드 실행
6. 회원 가입 폼을 POST 방식으로 데이터 처리
7. userService::create(회원정보) 메서드가 실행
8. userRepository::save(user) 메서드 되어 새로운 사용자 생성
9. DB에 내부적으로 INSERT 쿼리가 실행
10. 회원정보 생성

### 로그인

스프링 시큐리티를 통해 로그인을 수행하는 방법에는 여러 가지
- 가장 간단한 방법으로 SecurityConfig.java와 같은 시큐리티 설정 파일에 사용자 ID와 비밀번호를 직접 등록하여 인증을 처리하는 메모리 방식
- 이미 회원 가입을 통해 회원 정보를 DB에 저장했으므로 DB에서 회원 정보를 조회하여 로그인하는 방법을 사용할 것임
  - DB에서 사용자를 조회하는 서비스(UserSecurityService.java)를 만들고 그 서비스를 스프링 시큐리티에  등록함

### ENUM

ENUM은 코딩을 할 때 문자열 하드코딩 및 복사 작업에서 오는 실수를 예방하기 위해 고안된 특별한 클래스 입니다.
[강의](https://www.youtube.com/watch?v=HOCjdjZevGE&feature=youtu.be)

```java
public class Main {
  public static void main(String[] args) {
    /*
    UserRole ADMIN = new UserRole("ROLE_ADMIN"); // ADMIN("ROLE_ADMIN")
    UserRole USER = new UserRole("ROLE_USER"); // USER("ROLE_USER")
     */

    UserRole SUPER_ADMIN = UserRole.SUPER_ADMIN;
    UserRole ADMIN = UserRole.ADMIN;
    UserRole USER = UserRole.USER;

    System.out.println(SUPER_ADMIN);
    System.out.println(ADMIN);
    System.out.println(USER);

    System.out.println(UserRole.SUPER_ADMIN);
    System.out.println(UserRole.ADMIN);
    System.out.println(UserRole.USER);
  }
}

enum UserRole {
  SUPER_ADMIN("SUPER_ADMIN"),
  ADMIN("ROLE_ADMIN"),
  USER("ROLE_USER");

  /*
  public static UserRole SUPER_ADMIN = new UserRole("ROLE_SUPER_ADMIN"); // ADMIN("ROLE_ADMIN")
  public static UserRole ADMIN = new UserRole("ROLE_ADMIN"); // ADMIN("ROLE_ADMIN")
  public static UserRole USER = new UserRole("ROLE_USER"); // USER("ROLE_USER")
   */

  private UserRole(String value) {
    this.value = value;
  }

  private String value;

  @Override
  public String toString() {
    return "UserRole{" +
        "value='" + value + '\'' +
        '}';
  }
}
```







