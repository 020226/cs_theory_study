[강의자료](https://www.slog.gg/p/13959)\
[강의링크](https://www.youtube.com/watch?v=xI9_lTfwkXg&feature=youtu.be)

---
목차

---

### 00. QueryDSL 개요
사용 이유: 아래처럼 복잡한 쿼리는 JPA로 한계가 있다. + 다양한 SQL 언어 지원(유연함)
```sql
SELECT *
FROM question AS Q
LEFT JOIN site_user AS QA
ON Q.id = QA.author_id
WHERE (
    Q.subject LIKE "%검색어%"
    OR
    Q.content LIKE "%검색어%"
    OR
    QA.username LIKE "%검색어%"
)
```
#### 복잡한 쿼리는 QueryDSL로 쉽고 간결하게 작성 가능
- 동적 쿼리 생성
- 가독성이 높아짐
- 직관성 확보

### 01. QueryDSL 세팅

build.gradle
```
// querydsl 추가
buildscript {
	ext {
		queryDslVersion = "5.0.0"
	}
}

plugins {
	id 'java'
	id 'org.springframework.boot' version '3.4.1'
	id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.qsl'
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
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	compileOnly 'org.projectlombok:lombok'
	developmentOnly 'org.springframework.boot:spring-boot-devtools'
	runtimeOnly 'org.mariadb.jdbc:mariadb-java-client'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testRuntimeOnly 'org.junit.platform:junit-platform-launcher'

	//querydsl 추가
	// Spring boot 3.x이상에서 QueryDsl 패키지를 정의하는 방법
	implementation "com.querydsl:querydsl-jpa:${queryDslVersion}:jakarta"
	annotationProcessor "com.querydsl:querydsl-apt:${queryDslVersion}:jakarta"
	annotationProcessor "jakarta.annotation:jakarta.annotation-api"
	annotationProcessor "jakarta.persistence:jakarta.persistence-api"

	// P6Spy 의존성 추가
	implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.9.0'
}

tasks.named('test') {
	useJUnitPlatform()
}

// querydsl 추가 시작
def querydslDir = "$buildDir/generated/querydsl"

sourceSets {
	main.java.srcDirs += [ querydslDir ]
}

tasks.withType(JavaCompile) {
	options.annotationProcessorGeneratedSourcesDirectory = file(querydslDir)
}

clean.doLast {
	file(querydslDir).deleteDir()
}
```

application.properties
```
spring.application.name=qdsl
# DATABASE
spring.datasource.driverClassName=org.mariadb.jdbc.Driver
spring.datasource.url=jdbc:mariadb://127.0.0.1:3306/qsl?useUnicode=true&characterEncoding=utf8&autoReconnect=true&serverTimezone=Asia/Seoul
spring.datasource.username=root
spring.datasource.password=

# JPA
spring.jpa.hibernate.ddl-auto=create

# CONSOLE
spring.output.ansi.enabled=always
```

db/db.sql
```sql
# DB 삭제, 생성, 선택
DROP DATABASE IF EXISTS qsl;
CREATE DATABASE qsl;
USE qsl;
```

scr/main/java/../boundedContext/user/entity/SiteUser.java
```java
@Entity
@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@ToString
public class SiteUser {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(unique = true)
  private String username;

  private String password;

  @Column(unique = true)
  private String email;
}
```
#### Q클래스 생성

오른쪽 코끼리모양 Gradle -> Tasks -> other -> compileJava 클릭
성공 시, qsld/build/generated/querydsl/../QSiteUser 생성 완료
Q클래스가 있어야만 QueryDSL 사용 가능

- 스프링시큐리티는 내부적으로 User라는 클래스가 존재하여 클래스명을 SiteUser로 사용하였다

### 02. UserRepository 클래스의 계보
#### 리포지터리 계층 구조
|레벨|인터페이스/클래스|설명|
|------|---|---|
|최상위|Repository|최상위 인터페이스. 모든 리포지터리의 기반.|
|기본 CRUD 제공|CrudRepository|기본적인 save, findAll 등의 메서드 제공.|
|리스트 CRUD 제공|ListCrudRepository|리스트 형태 데이터 조회 메서드 확장.|
|JPA 기능 확장|JpaRepository|페이징, 정렬 등 JPA 관련 기능 추가.|
|사용자 정의 리포지터리|UserRepository|JpaRepository를 상속받아 사용자 정의.|

#### 커스텀 리포지터리 구성
|구성 요소|역할|
|------|---|
|UserRepositoryCustom|커스텀 쿼리를 정의하는 인터페이스.|
|UserRepositoryImpl|UserRepositoryCustom를 구현하며, QueryDSL을 사용.|

상속 관계_(i): interface
1. Repo(i) -> crudRepo(i) -> ListCrudRepo(i) -> JPARepo(i) -> UserRepo(i)
- Repo에서 saveAll, save, findAll 등을 가짐
- 인터페이스는 추상 클래스이다 보니 구현은 상속 받은 클래스에서 해야된다
- 그래서 상속 받은 인터페이스들이 각 구현을 담당하게 됨
- 하지만, JpaRepo를 상속받은 UserRepo는 왜 JpaRepo 메서드를 구현하지 않았나?
- = JPA가 알아서 JpaRepo의 구현을 해줌 = JpaRepo로부터 상속 받은 UserRepo가 직접 구현하지 않아도 된다
- JPA가 모르는 것은 따로 구현하긴 해야함

2. UserRepoCustom(i) -> UserRepo(i)
- JPA는 JpaRepo는 알고 있지만 UserRepoCustom은 모른다!
- UserRepoCustom가 있어야 QueryDSL을 사용할 수 있음
- UserRepoCustom 인터페이스에 구현한 메서드는 완성된 것이 아니기 때문에
- 그 구현체가 바로 UserRepoImpl이다. UserRepoImpl가 직접 구현.
- UserRepoCustom(i) -> UserRepoImpl(class) 관계가 되는 것
- = UserRepoCustom을 상속한 UserRepo가 구현하는 것이 아니라 UserRepoCustom을 UserRepoImpl가 상속받아 구현

즉, 내가 직접 만든 메서드 추가하고 싶다면 UserRepositoryCustom(i)에 추가 -> UserRepositoryImpl에 구현


### 03. UserRepository 세팅

#### entity에 새로운 속성을 적용 시 발생하는 에러
- 엔티티 클래스의 속성과 Q클래스의 속성이 매칭이 안 되었기 때문
- Gradle -> Tasks -> build -> clear 후 compileJava 클릭

/../boundedContext/user/repository/UserRepository
```java
// JPA 관련 담당 ex. findBy~
public interface UserRepository extends JpaRepository<SiteUser, Long>, UserRepositoryCustom { // SiteUser: 이름, Long: id 타입
}
```
/../boundedContext/user/repository/UserRepositoryCustom
```java
// 인터페이스명은 리포지터리이름+자유롭게(ex. Custom)
public interface UserRepositoryCustom {
  SiteUser getQslUser(Long id);
}
```
/../boundedContext/user/repository/UserRepositoryImpl
```java
// 클래스명은 리포지터리명+Impl이 규칙
// QueryDSL의 구현체로 QueryDSL 관련 코드만 작성
public class UserRepositoryImpl implements UserRepositoryCustom{
  @Override
  public SiteUser getQslUser(Long id) { // UserRepositoryCustom 생성 후 alt+shift+p로 메서드 넣어주기
    return null;
  }
}
```


Test 케이스
```java
@SpringBootTest
class QdslApplicationTests {
	@Autowired
	private UserRepository userRepository;

	@Test
	@DisplayName("회원 생성")
	void t1() { // {noop}1234 <- 비밀번호를 암호화하지 않도록(암호화는 {bcrypt}1234, id는 알아서 증가되기 때문에 null
		/*
		SiteUser u1 = new SiteUser(null, "user1", "{noop}1234", "user1@test.com");
		SiteUser u2 = new SiteUser(null, "user2", "{noop}1234", "user2@test.com");
        */

       SiteUser u1 = SiteUser.builder()
               .username("user1")
               .password("{noop}1234")
               .email("user1@test.com")
               .build();

       SiteUser u2 = SiteUser.builder()
               .username("user2")
               .password("{noop}1234")
               .email("user2@test.com")
               .build();


       userRepository.saveAll(Arrays.asList(u1, u2));
    }
}
```

### 04. QueryDSL 시작

#### QueryDSL 작업을 하기위해서 필요한 JPAQueryFactory 객체를 Bean 으로 등록

```sql
SELECT *
FROM site_user
WHERE id = 1
```
이라는 코드를 실행하고 싶을 때 JPA를 사용하면 userService.findById(1)을 사용했었음
QueryDSL을 활용하려면 **AppConfig 클래스** 추가


- @Configuration <= @Component
- @Bean = Ioc 컨테이너에 의해 관리하겠다 = 내부 싱글톤 패턴으로 관리하겠다
- jpaQueryFactory가 있어야 QueryDSL을 사용할 수 있다

/../qdsl/base/AppConfig
```java
// 설정 파일 = 규칙
@Configuration // @Configuration 해야 @Bean이 스캔이 됨
public class AppConfig {
  @Bean // 스프링 컨테이너에 Bean으로 등록된다.
  public JPAQueryFactory jpaQueryFactory(EntityManager entityManager) {
    return new JPAQueryFactory(entityManager);
  }
}
```

```java
// QSiteUser siteUser = QSiteUser.siteUser; 대신 import
import static com.sbs.exam.qdsl.boundedContext.user.entity.QSiteUser.siteUser;

@RequiredArgsConstructor // jpaQueryFactory를 요청이 있을 때마다 생성
public class UserRepositoryImpl implements UserRepositoryCustom{
  private final JPAQueryFactory jpaQueryFactory;
  @Override
  public SiteUser getQslUser(Long id) {
    /*
    SELECT *
    FROM site_user
    WHERE id = 1
    */
    return jpaQueryFactory
        .selectFrom(siteUser) // SELECT * FROM site_user
        .where(siteUser.id.eq(id)) // WHERE id = 1 // eq = equals
        .fetchOne(); // 단일 결과를 반환(없으면 null)
  }
}
```

#### UserRepositoryImpl::getQslUser에 대한 테스트케이스 구현

```java
@SpringBootTest
class QdslApplicationTests {
	@Autowired
	private UserRepository userRepository;

	@Test
	@DisplayName("회원 생성")
	void t1() { 
		SiteUser u1 = new SiteUser(null, "user1", "{noop}1234", "user1@test.com");
		SiteUser u2 = new SiteUser(null, "user2", "{noop}1234", "user2@test.com");

		userRepository.saveAll(Arrays.asList(u1, u2));
	}

	@Test
	@DisplayName("1번 회원을 Qsl로 가져오기")
	void t2() {
		// SELECT * FROM site_user WHERE id = 1;
		SiteUser u1 = userRepository.getQslUser(1L);

		assertThat(u1.getId()).isEqualTo(1L); // id가 1인지 확인
		assertThat(u1.getUsername()).isEqualTo("user1");
		assertThat(u1.getPassword()).isEqualTo("{noop}1234");
		assertThat(u1.getEmail()).isEqualTo("user1@test.com");
	}
}
```

### 05. initData

initData - 초기화 데이터
- 테스트용 샘플 데이터
- 개발용 샘플 데이터

기존 @BeforeEach나 @BeforeAll이 아닌 initData 쓰는 이유
- 테스트는 독립적으로 이뤄져야 한다.
1. 회원 테스트 데이터
  - 회원 가입
  - 회원 로그인
  - 회원 관련 등등
2. Article 테스트 데이터
   - 로그인한 회원이 게시물 작성

- 각각 테스트 데이터를 독립적으로 만들어주는 것은 번거로운 일이자
- '로그인한 회원이 게시물 작성'처럼 회원과 article이 연관된 상황이 있으면 두 테스트 모두 실행해야 함
- 그래서 initData를 도입!!

/../qdsl/base/TestInitData
```java
// @Configuration: @Bean과 함께 쓰여 bean에 붙어있는 메서드를 실행해서 객체를 만듦
// bean으로 만들어진 객체는 ioc컨테이너에 의해 관리된다
// @Profile({"dev","test"}): 이 클래스 안에 정의된 Bean들은 dev, test 모드에서만 실행됨
// Production(운영 환경, 라이브 서버)이 아니다
@Configuration
@Profile("test")
public class TestInitData { // 테스트 데이터를 초기화할 때 사용
  // CommandLineRunner : 주로 앱 실행 직후 초기데이터 세팅 및 초기화에 사용
  @Bean
  CommandLineRunner init(UserRepository userRepository) { // userRepository 의존성 주입
    return args -> { // 이 부분은 스프링부트 앱이 실행되고, 본격적으로 작동하기 전에 실행됨
      SiteUser u1 = SiteUser.builder()
          .username("user1")
          .password("{noop}1234")
          .email("user1@test.com")
          .build();
      SiteUser u2 = SiteUser.builder()
          .username("user2")
          .password("{noop}1234")
          .email("user2@test.com")
          .build();
      List<SiteUser> siteUsers = userRepository.saveAll(Arrays.asList(u1, u2));
    };
  }
}
```

- @Transactional을 붙여 각 테스트 케이스가 독립적인 환경에서 실행될 수 있도록
```java
@SpringBootTest
@Transactional // 각 테스트케이스에 전부 @Transactional을 붙인 효과
// @Test + @Transactional 조합은 자동으로 롤백을 유발시킨다.
@ActiveProfiles("test") // initData의 Profile("test")가 활성화됨
class QdslApplicationTests {
  @Autowired
  private UserRepository userRepository;

  @Test
  @DisplayName("회원 생성")
  void t1() {

    SiteUser u3 = SiteUser.builder()
        .username("user3")
        .password("{noop}1234")
        .email("user3@test.com")
        .build();

    SiteUser u4 = SiteUser.builder()
        .username("user4")
        .password("{noop}1234")
        .email("user4@test.com")
        .build();


    userRepository.saveAll(Arrays.asList(u3, u4));
  }
  // t2 생략
}
```

### 06. QueryDSL Q클래스 설명 추가
QueryDSL은 자바를 SQL처럼 구현한다!
QueryDSL을 사용하려면 **Entity 클래스를 기반으로** Q클래스를 생성해서 사용한다
- Q클래스 = QueryDSL을 자동으로 만들어주는 도우미 클래스
- Q + 엔티티 이름

### 07. QueryDSL 연습 테스트 케이스 구현

**실행되는 sql 쿼리를 알아야 QueryDSL을 검색해볼 수가 있다**
- ex. SELECT COUNT querydsl 검색

#### 전체 회원수 구하는 테스트 케이스 작성
```java
public interface UserRepositoryCustom {
  // 생략
  long getQslCount();
}
```
```java
@RequiredArgsConstructor
public class UserRepositoryImpl implements UserRepositoryCustom {
  // 생략
  @Override
  public long getQslCount() {
     // SELECT COUNT(*) FROM site_user
     return jpaQueryFactory
             .select(siteUser.count())
             .from(siteUser)
             .fetchOne();
  }
}
```
```java
@SpringBootTest
@Transactional 
@ActiveProfiles("test")
class QslTutorialApplicationTests {
  @Autowired
  private UserRepository userRepository;
  // 생략
  @Test
  @DisplayName("모든 회원 수")
  void t4() {
    long count = userRepository.getQslCount();

    assertThat(count).isGreaterThan(0); // 회원 수가 0보다 큰지 확인
  }
}
```
#### 가장 오래된 회원 테스트케이스 구현
```java
public interface UserRepositoryCustom {
  // 생략
  SiteUser getQslUserOrderByIdAscOne();
}
```
```java
@RequiredArgsConstructor
public class UserRepositoryImpl implements UserRepositoryCustom {
  // 생략
  @Override
  public SiteUser getQslUserOrderByIdAscOne() {
    /*
    SELECT *
    FROM site_user
    ORDER BY id ASC
    LIMIT 1;
    */
     return jpaQueryFactory
             .selectFrom(siteUser) // SELECT * FROM site_user
             .orderBy(siteUser.id.asc()) // ORDER BY id ASC
             .limit(1) // LIMIT 1
             .fetchOne(); // 단일 결과 하나를 반영
  }
}
```
```java
@SpringBootTest
@Transactional 
@ActiveProfiles("test")
class QslTutorialApplicationTests {
  @Autowired
  private UserRepository userRepository;
  // 생략
  @Test
  @DisplayName("가장 오래된 회원 1명")
  void t5() {
     SiteUser u1 = userRepository.getQslUserOrderByIdAscOne();
     assertThat(u1.getId()).isEqualTo(1L);
     assertThat(u1.getUsername()).isEqualTo("user1");
     assertThat(u1.getPassword()).isEqualTo("{noop}1234");
     assertThat(u1.getEmail()).isEqualTo("user1@test.com");
  }
}
```
#### 전체회원조회, 가장오래된순으로 정렬, 테스트케이스 구현
```java
public interface UserRepositoryCustom {
  // 생략
  List<SiteUser> getQslUserOrderByIdAsc();
}
```
```java
@RequiredArgsConstructor
public class UserRepositoryImpl implements UserRepositoryCustom {
  // 생략
  @Override
  public List<SiteUser> getQslUserOrderByIdAsc() {
     return jpaQueryFactory
             .selectFrom(siteUser) // SELECT * FROM site_user
             .orderBy(siteUser.id.asc()) // ORDER BY id ASC
             .fetch();
  }
}
```
```java
@SpringBootTest
@Transactional 
@ActiveProfiles("test")
class QslTutorialApplicationTests {
  @Autowired
  private UserRepository userRepository;
  // 생략
  @Test
  @DisplayName("전체 회원, 오래된 순")
  void t6() {
     List<SiteUser> users = userRepository.getQslUserOrderByIdAsc();
     SiteUser u1 = users.get(0);
     assertThat(u1.getId()).isEqualTo(1L);
     assertThat(u1.getUsername()).isEqualTo("user1");
     assertThat(u1.getPassword()).isEqualTo("{noop}1234");
     assertThat(u1.getEmail()).isEqualTo("user1@test.com");
     
     SiteUser u2 = users.get(1);
     assertThat(u2.getId()).isEqualTo(2L);
     assertThat(u2.getUsername()).isEqualTo("user2");
     assertThat(u2.getPassword()).isEqualTo("{noop}1234");
     assertThat(u2.getEmail()).isEqualTo("user2@test.com");
  }
}
```
#### 검색, 검색대상은 username과 email, List 리턴 테스트 케이스 구현
```sql
# 실행되어야 할 쿼리
SELECT *
FROM site_user
WHERE (
    username LIKE '%user1%'
    OR
    email LIKE '%user2@test.com%'
);
```

```java
public interface UserRepositoryCustom {
  // 생략
  List<SiteUser> searchQsl(String kw); // kw = keyword
}
```
```java
@RequiredArgsConstructor
public class UserRepositoryImpl implements UserRepositoryCustom {
  // 생략
  @Override
  public List<SiteUser> searchQsl(String kw) {
     return jpaQueryFactory
             .selectFrom(siteUser) // SELECT * FROM site_user
             .where(
                     siteUser.username.contains(kw)
                             .or(siteUser.email.contains(kw))
             ) // kw는 email이나 username 둘 중 하나가 들어오기 때문에
              // WHERE username = 'kw' 혹은 WHERE email = 'kw' 둘 중 하나의 쿼리 실행
             .fetch();
  }
}
```
```java
@SpringBootTest
@Transactional 
@ActiveProfiles("test")
class QslTutorialApplicationTests {
  @Autowired
  private UserRepository userRepository;
  // 생략
  @Test
  @DisplayName("검색, List 리턴, 검색 대상 : username, email")
  void t7() {
     // 검색 : username
     // username이 user1인 회원 검색
     List<SiteUser> users = userRepository.searchQsl("user1"); // WHERE
     assertThat(users.size()).isEqualTo(1); // 사이즈가 1
     SiteUser u = users.get(0); // 0번째 가져와라
     assertThat(u.getId()).isEqualTo(1L);
     assertThat(u.getUsername()).isEqualTo("user1");
     assertThat(u.getPassword()).isEqualTo("{noop}1234");
     assertThat(u.getEmail()).isEqualTo("user1@test.com");
     
     // 검색 : username
     // username이 user2인 회원 검색
     users = userRepository.searchQsl("user2");
     assertThat(users.size()).isEqualTo(1);
     u = users.get(0);
     assertThat(u.getId()).isEqualTo(2L);
     assertThat(u.getUsername()).isEqualTo("user2");
     assertThat(u.getPassword()).isEqualTo("{noop}1234");
     assertThat(u.getEmail()).isEqualTo("user2@test.com");
  }
}
```
#### 검색, 검색대상은 username과 email, Page 리턴, 테스트케이스 추가
한 페이지 당 보여줄 아이템 개수: 1
user1: 1페이지
user2: 2페이지

```java
public interface UserRepositoryCustom {
  // 생략
  Page<SiteUser> searchQsl(String kw, Pageable pageable);
}
```
```java
@RequiredArgsConstructor
public class UserRepositoryImpl implements UserRepositoryCustom {
  // 생략
  @Override
  public Page<SiteUser> searchQsl(String kw, Pageable pageable) { // kw를 받아서 페이징 처리
     // BooleanExpression : 검색 조건을 처리하는 객체
     // 검색 조건 (예: username, email 필드에서 keyword 포함 여부)
     // containsIgnoreCase : 대소문자를 구분하지 않는 검색을 수행
     BooleanExpression predicate = siteUser.username.containsIgnoreCase(kw)
             .or(siteUser.email.containsIgnoreCase(kw));

     // QueryDSL로 데이터 조회
     // QueryResults : 쿼리 실행 결과와 함께 페이징을 위한 추가 정보 포함
     QueryResults<SiteUser> queryResults = jpaQueryFactory
             .selectFrom(siteUser) // SELECT * FROM site_user
             .where(predicate) // WHERE username LIKE '%user%' OR email LIKE '%user%'
             .orderBy(siteUser.id.asc()) // ORDER BY id ASC
             .offset(pageable.getOffset()) // LIMIT {1}, ? // 페이지 시작 위치
             .limit(pageable.getPageSize()) // LIMIT ?, {1} // 페이지 크기
             .fetchResults(); // 데이터와 총 데이터 수를 가져옴 <- 총 데이터를 가져오기 때문에 데이터가 많아지면 문제가 될 수 있음(대체 방법 필요)

     // 결과와 totalCount를 기반으로 Page 객체 생성
     List<SiteUser> users = queryResults.getResults();
     long total = queryResults.getTotal(); // 총 데이터 수

     // PageImpl(page 관련 인터페이스) : 페이징 된 데이터와 메타데이터(전체 개수, 페이지 정보 등)을 포함
     return new PageImpl<>(users, pageable, total);
  }
}
```
```java
@SpringBootTest
@Transactional 
@ActiveProfiles("test")
class QslTutorialApplicationTests {
  @Autowired
  private UserRepository userRepository;
  // 생략
  @Test
  @DisplayName("검색, Page 리턴, id ASC, pageSize = 1, page = 0")
  void t8() {
     long totalCount = userRepository.count();
     int pageSize = 1; // 한 페이지에 보여줄 아이템 개수
     int totalPages = (int)Math.ceil(totalCount / (double)pageSize); // 페이지 소수점은 올림 처리
     int page = 1; // 0부터 시작하기 때문에 [현재 페이지] = 2번 째 페이지를 의미
     String kw = "user";

     List<Sort.Order> sorts = new ArrayList<>();
     sorts.add(Sort.Order.asc("id")); // id 기준 오름차순
     Pageable pageable = PageRequest.of(page, pageSize, Sort.by(sorts)); // 한 페이지당 몇 개까지 보여질 것인가
     Page<SiteUser> usersPage = userRepository.searchQsl(kw, pageable);

     assertThat(usersPage.getTotalPages()).isEqualTo(totalPages);
     assertThat(usersPage.getNumber()).isEqualTo(page);
     assertThat(usersPage.getSize()).isEqualTo(pageSize);

     List<SiteUser> users = usersPage.get().toList();
     assertThat(users.size()).isEqualTo(pageSize);

     SiteUser u = users.get(0);
     assertThat(u.getId()).isEqualTo(2L);
     assertThat(u.getUsername()).isEqualTo("user2");
     assertThat(u.getPassword()).isEqualTo("{noop}1234");
     assertThat(u.getEmail()).isEqualTo("user2@test.com");


     // 검색어 : user1
     // 한 페이지에 나올 수 있는 아이템 개수 : 1개
     // 정렬 : id 정순
     // 내용 가져오는 쿼리
		/*
		SELECT *
		FROM site_user
		WHERE username LIKE '%user%'
		OR email LIKE '%user%'
		ORDER BY id ASC
		LIMIT 1, 1; # LIMIT 1, 1은 첫 번째 게시물 건너뛰고 하나 보여줘. 
		*/

     // 쿼리가 두번 실행됨
     // 페이지네이션: 페이지 번호 구현을 위해
     // 전체 개수를 계산하는 SQL
		/*
		SELECT COUNT(*)
		FROM site_user
		WHERE username LIKE '%user%'
		OR email LIKE '%user%'
		*/
  }
}
```

##### t8 관련 실행되는 쿼리 정리
```sql
# 1. 전체 게시물 개수 카운트
SELECT COUNT(SU.*)
FROM site_user AS SU;

# 2. 검색 결과에 따른 게시물 카운트
SELECT COUNT(SU.*)
FROM site_user AS SU
WHERE (
  LOWER(SU.username) LIKE '%user%' ESCAPE '!'
  or 
  LOWER(SU.email) LIKE '%user%' ESCAPE '!';
);  

# 3. 검색 결과에 따른 게시물 조회
SELECT SU.* 
FROM site_user AS SU
WHERE (
  LOWER(SU.username) LIKE '%user%' ESCAPE '!'
  or 
  LOWER(SU.email) LIKE '%user%' ESCAPE '!';
)
ORDER BY SU.id LIMIT 1, 1; 
```

#### LIKE 검색의 특수기호, ESCAPE 문자
```sql
SELECT *
FROM site_user
WHERE username LIKE 'user!_good' ESCAPE '!';

WHERE username LIKE '100!%' ESCAPE '!'; # 출력결과 100%
WHERE username LIKE '#aaaa%' ESCAPE '#'; # 출력결과 aaaa
WHERE username LIKE 'user\_good' ESCAPE '\\'; # 출력결과 user good
```
!를 무시하고 user good으로 출력됨 
- 검색을 했을 때 특수기호가 검색이 안됨
- escape: 특수기호를 문자로 취급할 수 있기 위해 사용
- % : 다수 문자
- _ : 문자 1개 

#### 정렬 조건이 DESC에 따른 검색 결과 테스트 케이스 구현
searchQsl 메서드 하나로 ASC, DESC 구현

```java
@RequiredArgsConstructor
public class UserRepositoryImpl implements UserRepositoryCustom {
  private final JPAQueryFactory jpaQueryFactory;
  // 생략
   @Override
   public Page<SiteUser> searchQsl(String kw, Pageable pageable) {
      // BooleanExpression : 검색 조건을 처리하는 객체
      // 검색 조건 (예: username, email 필드에서 keyword 포함 여부)
      // containsIgnoreCase : 대소문자를 구분하지 않는 검색을 수행
      BooleanExpression predicate = siteUser.username.containsIgnoreCase(kw)
              .or(siteUser.email.containsIgnoreCase(kw));
      // QueryDSL로 데이터 조회
      // QueryResults : 쿼리 실행 결과와 함께 페이징을 위한 추가 정보 포함
      JPAQuery<SiteUser> usersQuery = jpaQueryFactory
              .selectFrom(siteUser) // SELECT * FROM site_user
              .where(predicate) // WHERE username LIKE '%user%' OR email LIKE '%user%'
              .offset(pageable.getOffset()) // LIMIT {1}, ? // 페이지 시작 위치
              .limit(pageable.getPageSize()); // LIMIT ?, {1} // 페이지 크기
            //.fatch 없이 반환값이 없어도 되도록 QueryResults<SiteUser> -> JPAQuery<SiteUser> 수정
      
      // pageable: 객체에 포함된 정렬 조건(sort)을 기반으로 동적 쿼리를 추가
      for (Sort.Order o : pageable.getSort()) { // Sort.Order: 각각의 정렬 조건
         // ORDER BY username
         // ORDER BY email
         PathBuilder pathBuilder = new PathBuilder(siteUser.getType(), siteUser.getMetadata());
         usersQuery.orderBy(new OrderSpecifier(o.isAscending() ? Order.ASC : Order.DESC, pathBuilder.get(o.getProperty())));
      }
      // 결과와 totalCount를 기반으로 Page 객체 생성
      List<SiteUser> users = usersQuery.fetch(); // usersQuery 결과를 반환하는 fetch를 users에 연결
      JPAQuery<Long> usersCountQuery = jpaQueryFactory
              .select(siteUser.count())
              .from(siteUser)
              .where(predicate);
      // PageImpl : 페이징 된 데이터와 메타데이터(전체 개수, 페이지 정보 등)을 포함
      return new PageImpl<>(users, pageable, usersCountQuery.fetchOne());   
   }
}
```
```java
@SpringBootTest
@Transactional 
@ActiveProfiles("test")
class QslTutorialApplicationTests {
  @Autowired
  private UserRepository userRepository;
  // 생략
  @Test
  @DisplayName("검색, Page 리턴, id DESC, pageSize = 1, page = 0")
  void t9() {
     long totalCount = userRepository.count();
     int pageSize = 1; // 한 페이지에 보여줄 아이템 개수
     int totalPages = (int)Math.ceil(totalCount / (double)pageSize); // 전체 페이지
     int page = 1; // 현재 페이지 -> 2번 째 페이지를 의미
     String kw = "user";
     
     // 페이징 처리 관련 코드
     List<Sort.Order> sorts = new ArrayList<>();
     sorts.add(Sort.Order.desc("id")); // id 기준 내림차순
     Pageable pageable = PageRequest.of(page, pageSize, Sort.by(sorts)); // 한 페이지당 몇 개까지 보여질 것인가
     Page<SiteUser> usersPage = userRepository.searchQsl(kw, pageable);
     
     assertThat(usersPage.getTotalPages()).isEqualTo(totalPages);
     assertThat(usersPage.getNumber()).isEqualTo(page);
     assertThat(usersPage.getSize()).isEqualTo(pageSize);
     
     List<SiteUser> users = usersPage.get().toList();
     assertThat(users.size()).isEqualTo(pageSize);
     // page 값이 1 == 2번째 페이지를 의미
     // 2번째 페이지는 1번 회원이 나와야 함
     SiteUser u = users.get(0);
     assertThat(u.getId()).isEqualTo(1L);
     assertThat(u.getUsername()).isEqualTo("user1");
     assertThat(u.getPassword()).isEqualTo("{noop}1234");
     assertThat(u.getEmail()).isEqualTo("user1@test.com");
  }
}
```

t9 실행된 쿼리
```sql
select *
where (
    lower(su1_0.username) like '%user%' escape '!'
    or
    lower(su1_0.email) like '%user%' escape '!'
)
order by su1_0.id.desc
limit 1,1;
```

####  페이지 자료구조에서는 왜 전체 엘리먼트 개수를 왜 필요로 할까?
`페이지 메뉴를 그려야 하기 때문에 필요하다.`

아래 쿼리는 UserRepositoryImpl의 searchQsl 중 
정렬이 끝난 뒤에 게시물 갯수 확인하는 쿼리

```sql
JPAQuery<Long> usersCountQuery = jpaQueryFactory
              .select(siteUser.count())
              .from(siteUser)
              .where(predicate);
```

페이지 자료구조에서는 왜 전체 엘리먼트 개수를 왜 필요로 할까? 
- `페이지 메뉴를 그려야 하기 때문에 필요하다.`

- pagesize : 5 => 한 화면에 보여질 개수
- page : 1 => 2번째 페이지
- 페이지네이션 표시(1~10)
- 예시) 게시물이 100개라면 페이지 전환이 이루어졌을 때
[1,10] 1-10까지 보여주고, [10, 10] 10개 건너뛰고 10개 보여주고, 
[20, 10] 20개 건너뛰고 10개 보여줌
- 끝 페이지 갯수를 파악하는 것이 중요(1~`10`) 끝 페이지가 10인 것을 알아야 11부터 보여줄 수 있기 때문.

- 총 게시물 수 : 100
- 한 페이지당 보여질 게시물 수 : 100
- 페이지 메뉴 : 1개


- 총 게시물 수 : 100
- 한 페이지당 보여질 게시물 수 : 10
- 페이지 메뉴 : 10개

- 페이지네이션 공식 = 총 게시물 수 / 한 페이지 당 보여질 개수
- 총 페이지 수 : 10
- 한 페이지당 보여질 게시물 개수 : 3
- 0p(3개), 1p(3개), 2p(3개), 3p(1개)
- 쿼리문: LIMIT 3, 1 -> LIMIT 6, 1 -> LIMIT 9, 1


### 8. 관심사 키워드 관련 테스트 코드

#### 회원에게 관심사 키워드 추가하는 테스트케이스 추가
`ManyToMany`를 이용해서 해결
- 중복 등록은 무시
- 엔티티 클래스 : InterestKeyword(interest_keyword 테이블)
- 중간테이블도 생성되어야 함(@ManyToMany)
- interest_keyword 테이블에 테니스, 오버워치, 헬스, 런닝에 해당하는 row 생성

interest_keyword 테이블에
1번 회원 관심사: 축구, 야구, 농구 /
2번 회원 관심사: 러닝, 테니스, 축구

interest_keyword 테이블에 1, 2번 중복 관심사 '축구'를 중복 저장하면 안됨
```java
@Entity
@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@ToString
public class InterestKeyword { // 위치 boundedContext.interestKeyword.InterestKeyword.java
  @Id
  private String content;
  
  // 아래 내용을 generate 해줌으로써 중복된 데이터가 들어가지 않도록 함
   @Override
   public boolean equals(Object o) {
      if (this == o) return true;
      if (o == null || getClass() != o.getClass()) return false;
      InterestKeyword that = (InterestKeyword) o;
      return Objects.equals(content, that.content);
   }
   @Override
   public int hashCode() {
      return Objects.hashCode(content);
   }
}
```
- interest_keyword 테이블에 중복된 축구 항목은 하나만 들어가야 한다
- 중복된 데이터가 들어가는 것이 아니라 하나의 데이터를 참조할 수 있도록
- interest_keyword 테이블 항목에도 1, 2, 3 등 id가 있을 것이고
- 회원 테이블에도 id가 존재하기 때문에
- 중간 테이블(site_user_interest_keywords)을 만들어 중복 데이터를 다루며 두 아이디를 참조하는 테이블 구성(2번 회원이 좋아하는 항목. 단, 중복 런닝은 제거된 채로 매칭.)

```java
@Entity
@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class SiteUser {
   @Id
   @GeneratedValue(strategy = GenerationType.IDENTITY)
   private Long id;

   @Column(unique = true)
   private String username;

   private String password;

   @Column(unique = true)
   private String email;
   
   // 중간테이블 만듦 
   @Builder.Default // 안 써주면 null이 들어감
   @ManyToMany(cascade =  CascadeType.ALL) // SiteUser와 생성, 소멸 시점을 같이함
   private Set<InterestKeyword> interestKeywords = new HashSet<>(); // 테스트 데이터를 만들 때 누락되어 있는 데이터임 = nullPointException 발생
   // @Builder.Default을 붙여주면 null이 아니라 비어있는 HashSet으로 초기화가 됨

   public void addInterestKeywordContent(String keywordContent) {
      interestKeywords.add(new InterestKeyword(keywordContent)); // new해서 객체를 담아준다
   }
}
```
```java
@SpringBootTest
@Transactional 
@ActiveProfiles("test")
class QslTutorialApplicationTests {
  @Autowired
  private UserRepository userRepository;
  // 생략
  @Test
  @DisplayName("회원에게 관심사를 등록할 수 있다.")
  @Rollback(false)
  void t10() {
    SiteUser u2 = userRepository.getQslUser(2L);
    u2.addInterestKeywordContent("테니스");
    u2.addInterestKeywordContent("오버워치");
    u2.addInterestKeywordContent("헬스");
    u2.addInterestKeywordContent("런닝");
    u2.addInterestKeywordContent("런닝"); // 중복 된 관심사 추가 -> 데이터 반영 x
     
     userRepository.save(u2);
    }
}
```

#### Set은 리스트와 비슷하지만 순서가 없고, 중복을 허용하지 않습니다. 
`대신 hashCode와 equals 메서드 오버라이드가 필수`

##### 숫자를 저장, 일반 데이터들은 자동으로 중복삽입금지 처리가 됨
```java
public class Main {
  public static void main(String[] args) {
    Set<Integer> number = new HashSet<>();
    number.add(1);
    number.add(2);
    number.add(3);
    number.add(4);
    number.add(4); // 중복 된 데이터는 삽입 되지 않는다.

    System.out.println(number);
  }
}
```
##### 일반객체들은 hashCode와 equals 메서드를 오버라이드 하지않는다면, 중복삽입금지 처리가 되지 않음
Generate 쉽게 생성하는 법
alt+insert 단축키 활용!!
```java
public class Main {
  public static void main(String[] args) {
    Set<InterestKeyword> interestKeywords = new HashSet<>();
    interestKeywords.add(new InterestKeyword("파스타"));
    interestKeywords.add(new InterestKeyword("파스타"));
    interestKeywords.add(new InterestKeyword("피자"));
    interestKeywords.add(new InterestKeyword("피자"));
    // 객체의 주소값이 다 다르기 때문에 중복 된 데이터가 삽입된다.

    System.out.println(interestKeywords.size());
    interestKeywords.stream().forEach(System.out::println);
  }
}

class InterestKeyword {
  private String content;

  public InterestKeyword(String content) {
    this.content = content;
  }

  @Override
  public String toString() {
    return "InterestKeyword{" +
        "content='" + content + '\'' +
        '}';
  }
}
```
##### Set의 올바른 사용예
- set을 사용할 때 hashCode, equals는 함께 사용해줘야 함!
- set은 리스트와 달리 순서가 존재하지 않는다
  - map처럼 키를 입력해서 데이터를 가져옴
```java
public class Main {
  public static void main(String[] args) {
    Set<InterestKeyword> interestKeywords = new HashSet<>();
    interestKeywords.add(new InterestKeyword("파스타")); // 동일한 데이터는
    interestKeywords.add(new InterestKeyword("파스타")); // 해시코드가 같다
    interestKeywords.add(new InterestKeyword("피자"));
    interestKeywords.add(new InterestKeyword("피자"));
    // 객체의 주소값이 다 다르기 때문에 중복 된 데이터가 삽입된다.

    System.out.println(interestKeywords.size());
    interestKeywords.stream().forEach(System.out::println);

    interestKeywords.stream().forEach(content -> {
      System.out.println(content.hashCode()); // equals를 통해 중복 데이터 해시코드 없이 해시코드 2개만 출력
    });
  }
}

class InterestKeyword {
  private String content;

  public InterestKeyword(String content) {
    this.content = content;
  }

  // 객체의 데이터가 동등한지 비교!
  @Override
  public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    InterestKeyword that = (InterestKeyword) o;
    return Objects.equals(content, that.content);
  }
    
  // 해당 객체의 해시코드를 반환
  @Override
  public int hashCode() {
    return Objects.hashCode(content);
  }

  @Override
  public String toString() {
    return "InterestKeyword{" +
        "content='" + content + '\'' +
        '}';
  }
}
```

#### 빌드시에 누락된 녀석이 null이 되는게 싫다면 @Builder.Default 추가해야 한다.
```java
public class Main {
  public static void main(String[] args) {
    // @Builder : 생성자 기반으로 객체를 생성!
    // 단점: 기본값으로 초기화된 필드가 무시될 수 있음

    // @Builder.Default : 필드의 기본값을 유지 시켜줌
    
    SiteUser user1 = SiteUser.builder() // @Builder 를 쓰면 builer 패턴을 쓸 수 있다
        // id를 넣지 않아도 빌더 패턴에 의해 auto_increment가 됨
         .username("user1")
        .password("{noop}1234")
        .email("user1@test.com")
         // @Builder.Default가 없다면 interestKeywords에 null이 들어간 상태 = interestKeywords를 쓰면 nullPointException이 나온다
         // @Builder.Default를 붙여주어 interestKeywords가 null이 아니라 비어있는 HashSet으로 초기화됨
            .build();

    // 위 아래는 사실상 같은 코드이다.
    // 위 코드에서는 null을 넣어도 @Builder.Default 있어서 초기화된 필드는 null처리 되지 않는다. 초기화 되어 interestKeywords = [] 처리 됨
     SiteUser user2 = new SiteUser(null, "user2", null, null, null);

    System.out.println(user1);
    System.out.println(user2);
  }
}

@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Builder
class InterestKeyword {
  String content;
}

@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@ToString
class SiteUser {
  private Long id;
  private String username;
  private String password;
  private String email;

  // 컬렉션 필드(map, set, list 등)의 값이 비어있으면 
  // @Builder.Default를 붙이지 않고 사용하면 nullPointException이 나오기 때문에 사용함 
   @Builder.Default
  private Set<InterestKeyword> interestKeywords = new HashSet<>();

  // 사용할 때 nullPointException이 나온다
  public void addInterestKeywordContent(String keywordContent) {
    interestKeywords.add(new InterestKeyword(keywordContent));
  }
}
```

#### 런닝에 관심있어하는 회원들 검색하는 테스트케이스 추가하기
- 처음할 때는 단계 별로 진행
  - JOIN부터 진행해서 조인이 되는지 확인하기

2번 회원의 관심있는 콘텐츠 목록(site_user_interest_keywords)
```sql
# QueryDSL에 의해 만들어 져야 하는 쿼리

SELECT SU.* # 회원만 조회, SUIK.interest_keywords_content를 붙이면 런닝도 나옴
FROM site_user AS SU
INNER JOIN site_user_interest_keywords AS SUIK 
ON SU.id = SUIK.site_user_id # site_user 중 site_user_interest_keywords 일치하는 것들 가져오기 위해 inner join
# 여기까지는 중간테이블 이너조인까지 완료한 것
# 테이블 [SU] <-> [중간테이블 SUIK] <-> [IK]
# SIUK에 중복 데이터가 들어갈 수 있음
# IK에는 중복 데이터가 들어가지 않음(콘텐츠 종류가 들어가기 때문에)
INNER JOIN interest_keyword AS IK
ON IK.content = SUIK.interest_keywords_content
WHERE IK.content = '런닝';
```
```java
public interface UserRepositoryCustom {
  // 생략
   List<SiteUser> getQslUserByInterestKeyword(String keyword);
}
```
##### QueryDSL로 SiteUser 엔티티와 InterestKeyword 엔티티 조인까지는 성공

```java
@RequiredArgsConstructor
public class UserRepositoryImpl implements UserRepositoryCustom {
  private final JPAQueryFactory jpaQueryFactory;
  // 생략
  @Override
  public List<SiteUser> getQslUserByInterestKeyword(String keyword) {
     return jpaQueryFactory
             .selectFrom(siteUser)
             .innerJoin(siteUser.interestKeywords) // INNER JOIN으로 중간 테이블까지 조인이 된다
             .fetch();
     /*
   t10까지 하고 dbeaver에서 실행하면 아래 쿼리 실행시킬 수 있음
   QueryDSL로 SiteUser 엔티티와 InterestKeyword 엔티티 조인까지는 성공
   SELECT SU.*
   FROM site_user AS SU
   INNER JOIN site_user_interest_keywords AS SUIK
   ON SU.id = SUIK.site_user_id;
     */
  }
}
```
##### Alias를 통해서 where 조건절에 조인 엔티티관련 조건 추가
Alias - 별칭 커스텀
- 보통 사용하지 않고 쿼리 dsl 사용하면 알아서 설정해줌
- 아래 코드는 Alias 사용하지 않고 실행함
```java
@RequiredArgsConstructor
public class UserRepositoryImpl implements UserRepositoryCustom {
  private final JPAQueryFactory jpaQueryFactory;
  // 생략
  @Override
  public List<SiteUser> getQslUserByInterestKeyword(String keyword) {
    // 별칭 커스텀(식별하기 위해 해본 것이지 보통 사용하진 않는다)
    // QSiteUser SU = QSiteUser.siteUser; // 별칭: su -> selectFrom(SU)
    // QInterestKeyword SUIK = QInterestKeyword.interestKeyword; // 별칭 : suik -> innerJoin(siteUser.interestKeywords, SUIK)
    return jpaQueryFactory
            .selectFrom(siteUser) // SELECT * FROM site_user AS SU
            .innerJoin(siteUser.interestKeywords, interestKeyword) // INNER JOIN site_user_interest_keywords AS SUIK
            .where(
                    interestKeyword.content.eq(keyword) // WHERE SUIK.content = keyword
                   )
             .fetch();
  }
}
```
```java
@SpringBootTest
@Transactional 
@ActiveProfiles("test")
class QslTutorialApplicationTests {
  @Autowired
  private UserRepository userRepository;
  // 생략
  @Test
  @DisplayName("런닝에 관심이 있는 회원들 검색")
  void t11() {
     List<SiteUser> users = userRepository.getQslUserByInterestKeyword("런닝");
     assertThat(users.size()).isEqualTo(1); // 회원이 하나인 상태
     SiteUser u = users.get(0);
     assertThat(u.getId()).isEqualTo(2L);
     assertThat(u.getUsername()).isEqualTo("user2");
     assertThat(u.getPassword()).isEqualTo("{noop}1234");
     assertThat(u.getEmail()).isEqualTo("user2@test.com");
  }
}
```

#### no qsl, 테니스에 관심있어하는 회원들 검색하는 테스트케이스 추가 후 구현
쿼리 dsl을 사용하지 않아도 jpa를 통해 내부적으로 동일한 쿼리가 사용됨
- 쿼리 dsl을 무조건적으로 사용하지 않아도 된다!
```java
@SpringBootTest
@Transactional 
@ActiveProfiles("test")
class QslTutorialApplicationTests {
  @Autowired
  private UserRepository userRepository;
  // 생략
  @Test
  @DisplayName("no QueryDSL, 테니스에 관심이 있는 회원들 검색")
  void t12() {
    // getQslUserByInterestKeyword를 사용하지 않고 jpa 문법 findBy 이용
     List<SiteUser> users = userRepository.findByInterestKeywords_content("테니스");
     assertThat(users.size()).isEqualTo(1);
     SiteUser u = users.get(0);
     assertThat(u.getId()).isEqualTo(2L);
     assertThat(u.getUsername()).isEqualTo("user2");
     assertThat(u.getPassword()).isEqualTo("{noop}1234");
     assertThat(u.getEmail()).isEqualTo("user2@test.com");
  }
}
```
커스텀이 아니라 UserRepository에 findByInterestKeywords_content를 만들어 no qsl
```java
public interface UserRepository extends JpaRepository<SiteUser, Long>, UserRepositoryCustom {
  List<SiteUser> findByInterestKeywords_content(String kw); // 찾는 쿼리는 쿼리 dsl을 시키지 않아도 
}
```

#### 1번 회원과 2번 회원이 공통 관심사 가질 수 있도록
testInitData 수정
```java
@Configuration
@Profile("test")
public class TestInitData {
   @Bean
   CommandLineRunner init(UserRepository userRepository) {
      return args -> {
         SiteUser u1 = SiteUser.builder()
                 .username("user1")
                 .password("{noop}1234")
                 .email("user1@test.com")
                 .build();

         SiteUser u2 = SiteUser.builder()
                 .username("user2")
                 .password("{noop}1234")
                 .email("user2@test.com")
                 .build();
         // 위 코드가 실행될 때 interest_keyword가 없을 수도 있다
         // 1. 위 코드를 저장한 뒤에 
         userRepository.saveAll(Arrays.asList(u1, u2));

         // SiteUser의 interestKeywords는 ManyToMany로 설정되어 있어서
         // 야구가 u1과 u2에 중복되지 않게 저장해야 하는데
         // 위에서 저장하지 않고 한 번에 아래 saveAll로 저장하면
         // 중복 저장이 되어 오류 발생!
         // ManyToMany로 회원이 있어야 키워드가 저장되기 때문
                  
         // 수정된 부분
         u1.addInterestKeywordContent("야구");
         u1.addInterestKeywordContent("농구");
         u2.addInterestKeywordContent("등산");
         u2.addInterestKeywordContent("캠핑");
         u2.addInterestKeywordContent("야구");
         // 2. interest_keyword를 저장해줘야
         userRepository.saveAll(Arrays.asList(u1, u2));
      };
   }
}
```

#### 팔로우 기능 테스트케이스 추가

```java
@Entity
@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class SiteUser {
   // 생략
   
   // set으로 만드는 이유: 나를 팔로우하는 동일한 한 사람이 둘이 될 수 없기 때문
   // u1이 시청자고 u2가 유튜버일 때 u1이 한 번 구독하면 또다시 구독할 수 없도록 set 사용
   @Builder.Default
   @ManyToMany(cascade = CascadeType.ALL) // 한 사람은 많은 사람을 구독할 수 있고, 많은 사람의 팔로워를 가질 수 있음
   private Set<SiteUser> followers = new HashSet<>();
   public void addInterestKeywordContent(String keywordContent) {
      interestKeywords.add(new InterestKeyword(keywordContent));
   }

   /*
   addFollower 대신 follow 함수 사용
   
   public void addFollower(SiteUser follower) {
      followers.add(follower); // 현재 사용자의 팔로워 목록에 새로운 팔로워 추가
      // site_user_followers 중간 테이블 생성됨(followers_id, site_user_id)
   }
   
   -> 현재 객체(this)의 관점에서 이루어짐. 현재 객체가 팔로워 목록(followers)에 새로운 사용자를 추가하는 방식
   = 유튜버가 구독자를 추가하는 방식임 - 틀린 방식!
   */
   public void follow(SiteUser following) {
     // follower: 나를 구독한 사람(시청자)
     // following: 내가 구독하는 사람(유튜버)
     following.getFollowers().add(this); // this = 사용자 = site_user
   }
   // -> 팔로우 객체의 팔로워 목록에 현재 객체를 추가하는 방식
   // 구독의 관점에서 구독자가 유튜버를 추가하는 방식임
}
```
```java
@SpringBootTest
@Transactional 
@ActiveProfiles("test")
class QslTutorialApplicationTests {
  @Autowired
  private UserRepository userRepository;
  // 생략
  @Test
  @DisplayName("u2=유튜버, u1=구독자 u1은 u2의 유튜브를 구독한다.")
  @Rollback(false)  void t13() {
     SiteUser u1 = userRepository.getQslUser(1L);
     SiteUser u2 = userRepository.getQslUser(2L);
     u1.follow(u2); // u1은 u2를 구독한다.
     userRepository.save(u2);
  }
}
```

#### 영속성 전이, CASCADE에 따라서 연관 엔티티가 같이 CRUD 될 수 있다.

`cascade 옵션`
: 연관된 엔티티에 대해서 저장, 삭제 등을 전파시킴 
- PERSIST, MERGE 등의 속성을 포함하는 `ALL`

키워드 입장: 1번 회원이 농구 관심 키워드 제거 -> cascade 옵션에 의해 농구 키워드도 삭제됨 = u1과 맵핑된 농구도 삭제됨

부모 엔티티, 자식 엔티티
- 부모 엔티티를 저장(삭제)할 때, 연관된 자식 엔티티도 함께 저장(삭제)

유튜버 u2와 u2를 구독하는 구독자 목록이 있고,
u1의 구독 하는 유튜버의 목록이 있을 때,
u2가 계정을 삭제하면 자동으로 u1의 구독 유튜버 목록에서도 삭제되도록 처리해줌.

#### @Transactional이 붙은 메서드 내에서는 경우에 따라서 save를 안해도 될 수도 있습니다.

JPA에서는 엔티티가 영속성 컨텍스트 안에 존재한다.

`영속성 컨텍스트`
: 엔티티의 1차 캐시 역할을 하는 메모리 공간. JPA가 사용하는 임시 작업 공간.
엔티티를 메모리에 올려 두고 관리하다가 @Transactional이 끝나는 순간 한꺼번에 데이터베이스에 반영한다.

- 데이터베이스에 바로 반영하는 것이 아니라
- Transactional은 작업을 하나로 묶어서 처리
- Transactional을 하는 동안 잠깐 머무른 공간으로 `영속성 컨텍스트`사용

```
@Service
class UserService() {
   // 영속 객체: JPA가 관리하는 객체
   // 비영속 객체: JPA가 아직 모르는 객체
   SiteUser u = new SiteUser("user1", "1234");
   
   user.Repository.save(u);
}
   
@Transactional // @Transactional이 붙음으로써 save 생략 가능 
public void create(SiteUser user, String email) { // -> 시작
   user.setEmail(email); // 더티 체킹
   // userRepository.save(u); 생략 가능
} // -> 끝
// 끝나고 나서 변경사항이 생겼을 때 jpa가 알아서 @Transactional에 의해 변경해줌
```
그럼에도 새로운 데이터(비영속 객체 등)를 저장(t1의 경우 - 없는 데이터 추가)할 때는 save를 해줘야 한다!
```java
@SpringBootTest
@Transactional 
@ActiveProfiles("test")
class QslTutorialApplicationTests {
  @Autowired
  private UserRepository userRepository;
  // 생략
  @Test
  @DisplayName("u2=유튜버, u1=구독자 u1은 u2의 유튜브를 구독한다.")
  @Rollback(false)  void t13() {
     SiteUser u1 = userRepository.getQslUser(1L);
     SiteUser u2 = userRepository.getQslUser(2L);
     u1.follow(u2); // u1은 u2를 구독한다. -> 변경사항을 감지하여 jpa가 알아서 save 됨
     // userRepository.save(u2); // @Transactional이 있기 때문에 save 안 해도 됨
  }
}
```

#### 트랜잭션이 시작되면, 해당 커넥션 내에서만 진행상황이 보여집니다.
검색 - 외래키 비활성화

트랜섹션을 통해 데이터를 생성하였더니 다른 Connection에서 데이터를 확인하면 데이터가 없어져있음
- 다른 커넥션의 데이터를 해당 커넥션에서 확인하기 위해서는 `커밋`(저장) 또는 (실패하면)`롤백`을 이용하면 된다!
```sql
# Connection: localhost 2
# DB 삭제, 생성, 선택
DROP DATABASE IF EXISTS qsl;
CREATE DATABASE qsl;
USE qsl;
    
SET foreign_key_checks = 0; # 외래키 비활성화
# DELETE 를 이용해서 삭제할 경우 1번 id를 삭제하면 id가 2번부터 생성됨
TRUNCATE interest_keyword; # TRUNCATE을 이용하면 삭제된 id 1번부터 다시 시작 가능
SET foreign_key_checks = 1; # 외래키 활성화

START TRANSACTION;                              
                              
INSERT INTO interest_keyword
SET content = '테니스';
INSERT INTO interest_keyword
SET content = '배구';

SELECT * FROM interest_keyword; # 농구가 들어와있지 않은 것을 확인 가능
```
```sql
# Connection: localhost
USE qsl;
SELECT * FROM interest_keyword; # 위의 커넥션에서 실행된 데이터 없음을 확인 가능
                              
INSERT INTO interest_keyword
SET content = '농구';
```
```java
@SpringBootTest
@Transactional // = START TRANSACTION;
@ActiveProfiles("test")
class QslTutorialApplicationTests {}
// 클래스가 끝나고 성공하면 COMMIT, 실패하면 ROLLBACK
```
---
```sql
# Connection: localhost 2
# DB 삭제, 생성, 선택
DROP DATABASE IF EXISTS qsl;
CREATE DATABASE qsl;
USE qsl;
    
SET foreign_key_checks = 0; # 외래키 비활성화
# DELETE 를 이용해서 삭제할 경우 1번 id를 삭제하면 id가 2번부터 생성됨
TRUNCATE interest_keyword; # TRUNCATE을 이용하면 삭제된 id 1번부터 다시 시작 가능
SET foreign_key_checks = 1; # 외래키 활성화

START TRANSACTION;                              
                              
INSERT INTO interest_keyword
SET content = '테니스';
INSERT INTO interest_keyword
SET content = '배구';

COMMIT; # 저장

SELECT * FROM interest_keyword; # 농구가 들어와있다
```

#### 트랜잭션이 시작되면 되면, 특정경우에 다른 커넥션에서 쓰기 Lock이 걸릴 수 있습니다.











































































