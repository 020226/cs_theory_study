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
  public SiteUser getQslUser(Long id) {
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
             .fetchResults(); // 데이터와 총 데이터 수를 가져옴

     // 결과와 totalCount를 기반으로 Page 객체 생성
     List<SiteUser> users = queryResults.getResults();
     long total = queryResults.getTotal(); // 총 데이터 수

     // PageImpl : 페이징 된 데이터와 메타데이터(전체 개수, 페이지 정보 등)을 포함
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

##### 실행되는 쿼리 정리
```sql
# 전체 게시물 개수 카운트
SELECT COUNT(SU.*)
FROM site_user AS SU;

# 검색 결과에 따른 게시물 카운트
SELECT COUNT(SU.*)
FROM site_user AS SU
WHERE (
  LOWER(SU.username) LIKE '%user%' ESCAPE '!'
  or 
  LOWER(SU.email) LIKE '%user%' ESCAPE '!';
);  

# 검색 결과에 따른 게시물 조회
SELECT SU.* 
FROM site_user AS SU
WHERE (
  LOWER(SU.username) LIKE '%user%' ESCAPE '!'
  or 
  LOWER(SU.email) LIKE '%user%' ESCAPE '!';
)
ORDER BY SU.id LIMIT 1, 1; 
```














