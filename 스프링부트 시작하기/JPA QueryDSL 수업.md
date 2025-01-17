[강의자료](https://www.slog.gg/p/13959)\
[강의링크](https://www.youtube.com/watch?v=xI9_lTfwkXg&feature=youtu.be)

---
목차

---

### 01. QueryDSL 개요
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

### 0. QueryDSL 세팅

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



















