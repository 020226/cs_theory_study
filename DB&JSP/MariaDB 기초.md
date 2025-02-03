## JDBC

[MVN REPOSITORY](https://mvnrepository.com/)

자바 프로그램 -> JDBC에 요청 -> JDBC가 MySQL과 소통

자바가 DBMS와 소통하는 창구가 JDBC임

pom.xml에 추가하여 JDBC 설치
```
<dependencies>
    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.30</version>
    </dependency>    
</dependencies>
```

## MySQL
#### 데이터베이스
```
- 생성 : CREATE DATABASE a1;
- 삭제 : DROP DATABASE IF EXISTS a1;
- 조회 : SHOW DATABASES;
```

#### 테이블
```
- 생성
CREATE TABLE article (
  title char(60),
  `body` TEXT
);
- 삭제 : DROP TABLE article;
- 조회 : SHOW TABLES

- 컬럼 추가
 - ALTER TABLE article ADD COLUMN id INT(10) FIRST;
- 컬럼 삭제
 - ALTER TABLE article DROP COLUMN id;
- 컬럼 수정
 - ALTER TABLE article MODIFY COLUMN id id INT(20);
```

#### 데이터
```
- 생성 : INSERT INTO article(title='제목1', `body`='내용1');
- 수정 : UPDATE article SET title='새 제목1' WHERE id = 2;
- 삭제 : DELETE FROM article WHERE id = 1;
- 조회 : SELECT * FROM article;
```
#### SELECT SQL 각 구문별 실행순서
1. FROM, JOIN
2. ON, WHERE
3. 2차 테이블 완성
4. GROUP BY
5. 그룹함수
6. 3차 테이블 완성(그룹작업이 있을 경우에만)
7. HAVING
8. 4차 테이블 완성(HAVING작업이 있을 경우에만)
9. ORDER BY
10. LIMIT
11. 고객(MySQL 클라이언트, 대표적으로 Sequel Pro, SQLYog, JDBC Driver 등)에게 전달


### 테이블 생성, 수정, 데이터 CRUD 
```sql
# 전체 데이터베이스 리스팅
SHOW DATABASES;

# `mysql` 데이터 베이스 선택
USE mysql;

# 테이블 리스팅
SHOW TABLES;

# 특정 테이블의 구조
DESCRIBE `event`;
DESC `event`;

# `test` 데이터 베이스 선택
USE test;

# 테이블 리스팅
SHOW TABLES;

# 기존에 a1 데이터베이스가 존재 한다면 삭제
DROP DATABASE IF EXISTS a1;

# 새 데이터베이스(`a1`) 생성
CREATE DATABASE a1;

# 데이터베이스(`a1`) 선택
USE a1;

# 데이터베이스 추가 되었는지 확인
SHOW DATABASES;

# 테이블 확인
SHOW TABLES;

# 게시물 테이블 article(title, body)을 만듭니다.
# VARCHAR(100) => 문자 100개 저장가능
# text => 문자 많이 저장가능
CREATE TABLE article (
	title VARCHAR(100),
	`body` TEXT
);

# 잘 추가되었는지 확인, 리스팅과 구조까지 확인
SHOW TABLES;
DESC article;

# 데이터 하나 추가(title = 제목, body = 내용)
INSERT INTO article
SET title = '제목',
`body` = '내용';

# 데이터 조회(title 만)
SELECT title
FROM article

# 데이터 조회(title, body)
SELECT title, `body`
FROM article

# 데이터 조회(body, title)
SELECT `body`, title
FROM article

# 데이터 조회(*)
SELECT *
FROM article

# 데이터 또 하나 추가(title = 제목, body = 내용)
INSERT INTO article
SET title = '제목',
`body` = '내용';

# 데이터 조회(*, 어떤게 2번 게시물인지 알 수 없음)
SELECT *
FROM article

# 테이블 구조 수정(id 칼럼 추가, first)
ALTER TABLE article ADD COLUMN id INT FIRST;

DESC article;

# 데이터 조회(*, id 칼럼의 값은 NULL)
SELECT *
FROM article

# 기존 데이터에 id값 추가(id = 1, id IS NULL)
UPDATE article
SET id = 1;

UPDATE article
SET id = 1
WHERE id IS NULL;

# 데이터 조회(*, 둘다 수정되어 버림..)
SELECT *
FROM article

# 기존 데이터 중 1개만 id를 2로 변경(LIMIT 1)
UPDATE article
SET id = 2
LIMIT 1;

# 데이터 조회(*)
SELECT *
FROM article

# 데이터 1개 추가(id = 3, title = 제목3, body = 내용3)
INSERT INTO article
SET id = 3,
title = '제목3',
`body` = '내용3';

# 데이터 조회(*)
SELECT *
FROM article

# 2번 게시물, 데이터 삭제 => DELETE
DELETE FROM article
WHERE id = 2;

# 데이터 조회(*)
SELECT *
FROM article

# 날짜 칼럼 추가 => regDate DATETIME
ALTER TABLE article ADD COLUMN regDate DATETIME AFTER id;

# 테이블 구조 확인
DESC article;

# 데이터 조회(*, 날짜 정보가 비어있음)
SELECT *
FROM article;

# 1번 게시물의 비어있는 날짜정보 채움(regDate = 2018-08-10 15:00:00)
UPDATE article
SET regDate = '2024-10-12 11:30:00'
WHERE id = 1;

# 데이터 조회(*, 이제 2번 게시물의 날짜 정보만 넣으면 됩니다.)
SELECT *
FROM article;

# NOW() 함수 실행해보기
SELECT NOW();

# 3번 게시물의 비어있는 날짜정보 채움(NOW())
UPDATE article
SET regDate = NOW()
WHERE id = 3;

# 데이터 조회(*)
SELECT *
FROM article;
```

### NOT NULL, auto_increment, LIKE, AND, OR, 칼럼이름, 칼럼순서 변경
```sql
# 기존에 a2 데이터베이스가 존재 한다면 삭제
DROP DATABASE IF EXISTS a2;
A
# 새 데이터베이스(`a2`) 생성
CREATE DATABASE a2;

# 새 데이터베이스(`a2`) 선택
USE a2;

# article 테이블 생성(id, regDate, title, body)
CREATE TABLE article (
	id INT,
	regDate DATETIME,
	title CHAR(100),
	`body` TEXT
);

# article 테이블 조회(*)
SELECT *
FROM article;

# article 테이블에 data insert (regDate = NOW(), title = '제목', body = '내용')
INSERT INTO article
SET regDate = NOW(),
title = '제목',
`body` = '내용';

# article 테이블에 data insert (regDate = NOW(), title = '제목', body = '내용')
INSERT INTO article
SET regDate = NOW(),
title = '제목',
`body` = '내용';

# article 테이블 조회(*)
SELECT * FROM article;

## id가 NULL인 데이터 생성이 가능하네?

# id 데이터는 꼭 필수 이기 때문에 NULL을 허용하지 않게 바꾼다.(alter table, not null)
## 기존의 NULL값 때문에 경고가 뜬다.
## 기존의 NULL값이 0으로 바뀐다.
ALTER TABLE article MODIFY COLUMN id INT NOT NULL; # null 허용하지 않음

SELECT * FROM article;
# article 테이블 조회(*)

# 생각해 보니 모든 행(row)의 id 값은 유니크 해야한다.(ADD PRIMARY KEY(id))
## 오류가 난다. 왜냐하면 기존의 데이터 중에서 중복되는게 있기 때문에
ALTER TABLE article ADD PRIMARY KEY(id);

# id가 0인 것 중에서 1개를 id 1로 바꾼다.
SELECT * FROM article;

UPDATE article
SET id = 1
WHERE id = 0
LIMIT 1;

# article 테이블 조회(*)
SELECT * FROM article;

# id가 0인것을 id 2로 바꾼다.
UPDATE article
SET id = 2
WHERE id = 0;

SELECT * FROM article;
# 생각해 보니 모든 행(row)의 id 값은 유니크 해야한다.(ADD PRIMARY KEY(id))
## 이제 적용이 잘 된다.
ALTER TABLE article ADD PRIMARY KEY(id);

DESC article;
# id 칼럼에 auto_increment 를 건다.
## auto_increment 를 걸기전에 해당 칼럼은 무조건 key 여야 한다.
INSERT INTO article
SET regDate = NOW(),
title = '제목',
`body` = '내용';

# article 테이블 구조확인(desc)
DESC article;

SELECT * FROM article;

UPDATE article
SET id = 3
WHERE id = 0;

INSERT INTO article
SET id = 4,
regDate = NOW(),
title = '제목',
`body` = '내용';

ALTER TABLE article MODIFY COLUMN id INT NOT NULL AUTO_INCREMENT;

# AUTO_INCREMENT은 PRIMARY_KEY가 걸려있어야지만 사용 가능!

INSERT INTO article
SET regDate = NOW(),
title = '제목',
`body` = '내용';

SELECT * FROM article;

DESC article;
# 나머지 칼럼 모두에도 not null을 적용해주세요.
ALTER TABLE article MODIFY COLUMN regDate DATETIME NOT NULL;
ALTER TABLE article MODIFY COLUMN title CHAR(100) NOT NULL;
ALTER TABLE article MODIFY COLUMN `body` TEXT NOT NULL;

DESC article;
# id 칼럼에 UNSIGNED 속성을 추가하세요.
# UNSIGNED: 음수 값 금지
ALTER TABLE article MODIFY COLUMN id INT UNSIGNED NOT NULL AUTO_INCREMENT;

DESC article;
# 작성자(writer) 칼럼을 title 칼럼 다음에 추가해주세요.
ALTER TABLE article ADD COLUMN writer VARCHAR(100) NOT NULL AFTER title;

DESC article;

# 작성자(writer) 칼럼의 이름을 nickname 으로 변경해주세요.(ALTER TABLE article CHANGE oldName newName TYPE 조건)
DESC article;

# MODIFY로 컬럼의 이름을 바꿀 수 없음 -> CHANGE
ALTER TABLE article CHANGE writer nickname VARCHAR(100) NOT NULL;

DESC article;

# nickname 칼럼의 위치를 body 밑으로 보내주세요.(MODIFY COLUMN nickname)
ALTER TABLE article MODIFY COLUMN nickname VARCHAR(100) NOT NULL AFTER `body`;

DESC article;
# hit 조회수 칼럼 추가 한 후 삭제해주세요.
ALTER TABLE article ADD COLUMN hit INT UNSIGNED NOT NULL AFTER nickname;

ALTER TABLE article DROP COLUMN hit;

DESC article;

# hit 조회수 칼럼을 다시 추가
ALTER TABLE article ADD COLUMN hit INT UNSIGNED NOT NULL AFTER nickname;

# 기존의 비어있는 닉네임 채워넣기(무명)
SELECT * FROM article;

UPDATE article
SET nickname = '무명'
WHERE nickname = '';

SELECT * FROM article;

# article 테이블에 데이터 추가(regDate = NOW(), title = '제목3', body = '내용3', nickname = '홍길순', hit = 10)
INSERT INTO article
SET regDate = NOW(),
title = '제목3',
`body` = '내용3',
nickname = '홍길순',
hit = 10;

# article 테이블에 데이터 추가(regDate = NOW(), title = '제목4', body = '내용4', nickname = '홍길동', hit = 55)
INSERT INTO article
SET regDate = NOW(),
title = '제목4',
`body` = '내용4',
nickname = '홍길동',
hit = 55;

# article 테이블에 데이터 추가(regDate = NOW(), title = '제목5', body = '내용5', nickname = '홍길동', hit = 10)
INSERT INTO article
SET regDate = NOW(),
title = '제목5',
`body` = '내용5',
nickname = '홍길동',
hit = 10;

# article 테이블에 데이터 추가(regDate = NOW(), title = '제목6', body = '내용6', nickname = '임꺽정', hit = 100)
INSERT INTO article
SET regDate = NOW(),
title = '제목6',
`body` = '내용6',
nickname = '임꺽정',
hit = 100;

SELECT * FROM article;
# 조회수 가장 많은 게시물 3개 만 보여주세요., 힌트 : ORDER BY, LIMIT
SELECT *
FROM article
ORDER BY hit;

SELECT *
FROM article
ORDER BY hit ASC;

SELECT *
FROM article
ORDER BY hit DESC;

SELECT *
FROM article
ORDER BY hit DESC
LIMIT 3;

# SELECT > ORDER > LIMIT

# 작성자명이 '홍길'로 시작하는 게시물만 보여주세요., 힌트 : LIKE '홍길%'
SELECT *
FROM article
WHERE nickname LIKE '홍길%';

SELECT *
FROM article
WHERE nickname LIKE '%홍길';

SELECT *
FROM article
WHERE nickname LIKE '%홍길%';

# '홍길%' : '홍길'로 시작하는
# '%홍길' : '홍길'로 끝나는
# '%홍길%' : '홍길'이라는 단어를 포함하는

# 조회수가 10 이상 55 이하 인것만 보여주세요., 힌트 : WHERE 조건1 AND 조건2
SELECT *
FROM article
WHERE hit >= 10
AND hit <= 55;

# 정답
SELECT *
FROM article
WHERE hit BETWEEN 10 AND 55;

SELECT *
FROM article
WHERE hit BETWEEN 10 AND 55
ORDER BY hit;

# 작성자가 '무명'이 아니고 조회수가 50 이하인 것만 보여주세요., 힌트 : !=
SELECT *
FROM article
WHERE nickname != '무명'
AND hit <= 55;

# 작성자가 '무명' 이거나 조회수가 55 이상인 게시물을 보여주세요. 힌트 : OR
SELECT *
FROM article
WHERE nickname != '무명'
OR hit <= 55;
```

### INNER JOIN, 사원, 부서
```sql
# a5 데이터베이스 삭제/생성/선택
DROP DATABASE IF EXISTS a5;
CREATE DATABASE a5;
USE a5;

# 부서(dept) 테이블 생성 및 홍보부서 기획부서 추가
CREATE TABLE dept (
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	regDate DATETIME NOT NULL,
	`name` CHAR(100) NOT NULL UNIQUE
);

INSERT INTO dept
SET regDate = NOW(),
`name` = '홍보';

INSERT INTO dept
SET regDate = NOW(),
`name` = '기획';

SELECT * FROM dept;

# 사원(emp) 테이블 생성 및 홍길동사원(홍보부서), 홍길순사원(홍보부서), 임꺽정사원(기획부서) 추가
CREATE TABLE emp (
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	regDate DATETIME NOT NULL,
	`name` CHAR(100) NOT NULL,
	deptName CHAR(100) NOT NULL
);

INSERT INTO emp
SET regDate = NOW(),
`name` = '홍길동',
deptName = '홍보';

INSERT INTO emp
SET regDate = NOW(),
`name` = '홍길순',
deptName = '홍보';

INSERT INTO emp
SET regDate = NOW(),
`name` = '임꺽정',
deptName = '기획';

SELECT * FROM emp;

# 홍보를 마케팅으로 변경
SELECT * FROM dept;
SELECT * FROM emp;

UPDATE dept
SET `name` = '마케팅'
WHERE `name` = '홍보';

SELECT * FROM dept;

UPDATE emp
SET deptName = '마케팅'
WHERE deptName = '홍보';

SELECT * FROM emp;
# 마케팅을 홍보로 변경
UPDATE dept
SET `name` = '홍보'
WHERE `name` = '마케팅';

SELECT * FROM dept;

UPDATE emp
SET deptName = '홍보'
WHERE deptName = '마케팅';

SELECT * FROM emp;

# 홍보를 마케팅으로 변경
UPDATE dept
SET `name` = '마케팅'
WHERE `name` = '홍보';

SELECT * FROM dept;

UPDATE emp
SET deptName = '마케팅'
WHERE deptName = '홍보';

SELECT * FROM emp;

# 구조를 변경하기로 결정(사원 테이블에서, 이제는 부서를 이름이 아닌 번호로 기억)
SELECT * FROM emp;

ALTER TABLE emp ADD COLUMN deptId INT UNSIGNED NOT NULL;

SELECT * FROM emp;
SELECT * FROM dept;

# deptName이 '마케팅' 부서의 deptId를 1로 변경
UPDATE emp
SET deptId = 1
WHERE deptName = '마케팅';

# deptName이 '기획' 부서의 deptId를 2로 변경
UPDATE emp
SET deptId = 2
WHERE deptName = '기획';

SELECT * FROM emp;

# emp테이블에 deptName을 제거
ALTER TABLE emp DROP deptName;

# 사장님께 드릴 인명록을 생성
SELECT * FROM emp;

# 사장님께서 부서번호가 아니라 부서명을 알고 싶어하신다.
# 그래서 dept 테이블 조회법을 알려드리고 혼이 났다.
SELECT *
FROM dept
WHERE id = 1;

# 사장님께 드릴 인명록을 생성(v2, 부서명 포함, ON 없이)
# 이상한 데이터가 생성되어서 혼남
SELECT *
FROM emp, dept;

SELECT *
FROM emp
INNER JOIN dept;

# 사장님께 드릴 인명록을 생성(v3, 부서명 포함, 올바른 조인 룰(ON) 적용)
# 보고용으로 좀 더 편하게 보여지도록 고쳐야 한다고 지적받음
SELECT *
FROM emp
INNER JOIN dept
ON emp.deptId = dept.id;

SELECT dept.*
FROM emp
INNER JOIN dept
ON emp.deptId = dept.id;

SELECT emp.*, dept.name
FROM emp
INNER JOIN dept
ON emp.deptId = dept.id;

# 사장님께 드릴 인명록을 생성(v4, 사장님께서 보시기에 편한 칼럼명(AS))
SELECT emp.*,
dept.name AS `부서명`
FROM emp
INNER JOIN dept
ON emp.deptId = dept.id;

SELECT emp.id AS `사원번호`,
DATE(emp.regDate) AS `입사일`,
emp.name AS `사원명`,
dept.name AS `부서명`
FROM emp
INNER JOIN dept
ON emp.deptId = dept.id;

# 사장님께 드릴 인명록을 생성(v5, 테이블 AS 적용)
SELECT emp.id AS `사원번호`,
DATE(emp.regDate) AS `입사일`,
emp.name AS `사원명`,
dept.name AS `부서명`
FROM emp
INNER JOIN dept
ON emp.deptId = dept.id
ORDER BY `부서명`;

SELECT emp.id AS `사원번호`,
DATE(emp.regDate) AS `입사일`,
emp.name AS `사원명`,
dept.name AS `부서명`
FROM emp
INNER JOIN dept
ON emp.deptId = dept.id
ORDER BY `부서명`, `사원명` DESC;

# 실무 수준 코드
SELECT E.id AS `사원번호`,
DATE(E.regDate) AS `입사일`,
E.name AS `사원명`,
D.name AS `부서명`
FROM emp AS E
INNER JOIN dept AS D
ON E.deptId = D.id
ORDER BY `부서명`, `사원명` DESC;
```

### SUM, MAX, MIN, COUNT, GROUP BY
```sql
# a6 DB 삭제/생성/선택
DROP DATABASE IF EXISTS a6;
CREATE DATABASE a6;
USE a6;

# 부서(홍보, 기획)
CREATE TABLE dept (
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	regDate DATETIME NOT NULL,
	`name` CHAR(100) NOT NULL UNIQUE
);

INSERT INTO dept
SET regDate = NOW(),
`name` = '홍보';

INSERT INTO dept
SET regDate = NOW(),
`name` = '기획';

SELECT * FROM dept;

# 사원(홍길동/홍보/5000만원, 홍길순/홍보/6000만원, 임꺽정/기획/4000만원)
CREATE TABLE emp (
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	regDate DATETIME NOT NULL,
	`name` CHAR(100) NOT NULL,
	deptId INT UNSIGNED NOT NULL,
	salary INT UNSIGNED NOT NULL
);

INSERT INTO emp
SET regDate = NOW(),
`name` = '홍길동',
deptId = 1,
salary = 5000;

INSERT INTO emp
SET regDate = NOW(),
`name` = '홍길순',
deptId = 1,
salary = 6000;

INSERT INTO emp
SET regDate = NOW(),
`name` = '임꺽정',
deptId = 2,
salary = 4000;

# 사원 수 출력
SELECT *, NOW(), YEAR(NOW()), MONTH(NOW())
FROM emp;

SELECT *, CONCAT('안', '녕')
FROM emp;

SELECT COUNT(*)
FROM emp;

# 가장 큰 사원 번호 출력
SELECT MAX(id)
FROM emp;

SELECT MIN(id)
FROM emp;

# 가장 고액 연봉
SELECT MAX(salary)
FROM emp;

# 가장 저액 연봉
SELECT MIN(salary)
FROM emp;

# 회사에서 1년 고정 지출(인건비)
SELECT * FROM emp;

SELECT SUM(salary)
FROM emp;

# 부서별, 1년 고정 지출(인건비)
SELECT deptId, SUM(salary)
FROM emp
GROUP BY deptId;

# GROUP BY -> SUM

# 부서별, 최고연봉
SELECT deptId, MAX(salary)
FROM emp
GROUP BY deptId;

# 부서별, 최저연봉
SELECT deptId, MIN(salary)
FROM emp
GROUP BY deptId;

# 부서별, 평균연봉
SELECT deptId, AVG(salary)
FROM emp
GROUP BY deptId;

# 부서별, 부서명, 사원리스트, 평균연봉, 최고연봉, 최소연봉, 사원수 
## V1(조인 안한 버전)
SELECT E.deptId AS `부서번호`,
GROUP_CONCAT(E.`name`) AS `사원리스트`,
AVG(E.salary) AS `평균연봉`,
MAX(E.salary) AS `최고연봉`,
MIN(E.salary) AS `최저연봉`,
COUNT(*) AS `사원수`
FROM emp AS E, dept AS D
GROUP BY E.deptId;

## V2(조인해서 부서명까지 나오는 버전)
SELECT *
FROM emp
INNER JOIN dept
ON emp.deptId = dept.id;

SELECT D.`name` AS `부서명`,
GROUP_CONCAT(E.`name`) AS `사원리스트`,
AVG(E.salary) AS `평균연봉`,
MAX(E.salary) AS `최고연봉`,
MIN(E.salary) AS `최저연봉`,
COUNT(*) AS `사원수`
FROM emp AS E
INNER JOIN dept AS D
ON E.deptId = D.id
GROUP BY E.deptId;

SELECT D.`name` AS `부서명`,
GROUP_CONCAT(E.`name`) AS `사원리스트`,
TRUNCATE(AVG(E.salary), 0) AS `평균연봉`,
MAX(E.salary) AS `최고연봉`,
MIN(E.salary) AS `최저연봉`,
COUNT(*) AS `사원수`
FROM emp AS E
INNER JOIN dept AS D
ON E.deptId = D.id
GROUP BY E.deptId;

SELECT D.`name` AS `부서명`,
GROUP_CONCAT(E.`name`) AS `사원리스트`,
CONCAT(TRUNCATE(AVG(E.salary), 0), '만원') AS `평균연봉`,
MAX(E.salary) AS `최고연봉`,
MIN(E.salary) AS `최저연봉`,
COUNT(*) AS `사원수`
FROM emp AS E
INNER JOIN dept AS D
ON E.deptId = D.id
GROUP BY E.deptId;

## V3(V2에서 평균연봉이 5000이상인 부서로 추리기)
# 아래 쿼리는 에러가 나온다.
# WHERE 절에서 에러 발생!!
'''
SELECT D.`name` AS `부서명`,
GROUP_CONCAT(E.`name`) AS `사원리스트`,
TRUNCATE(AVG(E.salary), 0) AS `평균연봉`,
MAX(E.salary) AS `최고연봉`,
MIN(E.salary) AS `최저연봉`,
COUNT(*) AS `사원수`
FROM emp AS E
INNER JOIN dept AS D
ON E.deptId = D.id
WHERE `평균연봉` >= 5000
GROUP BY E.deptId;
'''

# FROM -> WHERE -> GROUP -> 그룹함수

SELECT D.`name` AS `부서명`,
GROUP_CONCAT(E.`name`) AS `사원리스트`,
TRUNCATE(AVG(E.salary), 0) AS `평균연봉`,
MAX(E.salary) AS `최고연봉`,
MIN(E.salary) AS `최저연봉`,
COUNT(*) AS `사원수`
FROM emp AS E
INNER JOIN dept AS D
ON E.deptId = D.id
GROUP BY E.deptId
HAVING `평균연봉` >= 5000;

# FROM -> WHERE -> GROUP -> 그룹함수 -> HAVING -> ORDER BY -> LIMIT

## V4(V3에서 HAVING 없이 서브쿼리로 수행)
### HINT, UNION을 이용한 서브쿼리
# SELECT *
# FROM (
#     select 1 AS id
#     union
#     select 2
#     UNION
#     select 3
# ) AS A

# UNION : 중복제거 후 합쳐줌
# UNION ALL : 중복을 제거하지 않고 합쳐준다.

SELECT *
FROM (
	SELECT *
	FROM (
		SELECT id + 1 AS id, `name`
		FROM emp
	) AS E
	WHERE E.id > 3
) AS E
WHERE E.id = 4;


SELECT *
FROM (
	SELECT D.`name` AS `부서명`,
	GROUP_CONCAT(E.`name`) AS `사원리스트`,
	TRUNCATE(AVG(E.salary), 0) AS `평균연봉`,
	MAX(E.salary) AS `최고연봉`,
	MIN(E.salary) AS `최저연봉`,
	COUNT(*) AS `사원수`
	FROM emp AS E
	INNER JOIN dept AS D
	ON E.deptId = D.id
	GROUP BY E.deptId
) AS D
WHERE D.`평균연봉` >= 5000;
```

### LEFT JOIN, UNION
```sql
# a6 DB 삭제/생성/선택
DROP DATABASE IF EXISTS a6;
CREATE DATABASE a6;
USE a6;

# 부서(홍보, 기획, IT)
CREATE TABLE dept (
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	regDate DATETIME NOT NULL,
	`name` CHAR(100) NOT NULL UNIQUE
);

INSERT INTO dept
SET regDate = NOW(),
`name` = '홍보';

INSERT INTO dept
SET regDate = NOW(),
`name` = '기획';

INSERT INTO dept
SET regDate = NOW(),
`name` = 'IT';

SELECT * FROM dept;

# 사원(홍길동/홍보/5000만원, 홍길순/홍보/6000만원, 임꺽정/기획/4000만원)
CREATE TABLE emp (
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	regDate DATETIME NOT NULL,
	`name` CHAR(100) NOT NULL,
	deptId INT UNSIGNED NOT NULL,
	salary INT UNSIGNED NOT NULL
);

INSERT INTO emp
SET regDate = NOW(),
`name` = '홍길동',
deptId = 1,
salary = 5000;

INSERT INTO emp
SET regDate = NOW(),
`name` = '홍길순',
deptId = 1,
salary = 6000;

INSERT INTO emp
SET regDate = NOW(),
`name` = '임꺽정',
deptId = 2,
salary = 4000;

SELECT * FROM emp;

## IT부서는 아직 사원이 없음

# 전 사원에 대하여, [부서명, 사원번호, 사원명] 양식으로 출력
SELECT D.`name` AS `부서명`,
E.id AS `사원번호`,
E.`name` AS `사원명`
FROM emp AS E
INNER JOIN dept AS D
ON E.deptId = D.id;

# 전 사원에 대하여, [부서명, 사원번호, 사원명] 양식으로 출력
## IT부서는 [IT, NULL, NULL] 으로 출력
SELECT D.`name` AS `부서명`,
E.id AS `사원번호`,
E.`name` AS `사원명`
FROM dept AS D
LEFT JOIN emp AS E
ON E.deptId = D.id;

# 전 사원에 대하여, [부서명, 사원번호, 사원명] 양식으로 출력
## IT부서는 [IT, 0, -] 으로 출력
SELECT D.`name` AS `부서명`,
IF(E.id IS NOT NULL, E.id, 0) AS `사원번호`,
IFNULL(E.`name`, "-") AS `사원명`
FROM dept AS D
LEFT JOIN emp AS E
ON E.deptId = D.id;

# 모든 부서별, 최고연봉, IT부서는 0원으로 표시
SELECT D.`name` AS `부서명`,
IFNULL(MAX(E.salary), 0) `최고연봉`
FROM dept AS D
LEFT JOIN emp AS E
ON E.deptId = D.id
GROUP BY D.id;

SELECT D.`name` AS `부서명`,
MAX(IFNULL(E.salary, 0)) `최고연봉`
FROM dept AS D
LEFT JOIN emp AS E
ON E.deptId = D.id
GROUP BY D.id;

# 모든 부서별, 최저연봉, IT부서는 0원으로 표시
SELECT D.`name` AS `부서명`,
MIN(IFNULL(E.salary, 0)) `최고연봉`
FROM dept AS D
LEFT JOIN emp AS E
ON E.deptId = D.id
GROUP BY D.id;

# 모든 부서별, 평균연봉, IT부서는 0원으로 표시
SELECT D.`name` AS `부서명`,
TRUNCATE(AVG(IFNULL(E.salary, 0)), 0) `최고연봉`
FROM dept AS D
LEFT JOIN emp AS E
ON E.deptId = D.id
GROUP BY D.id;

# 하나의 쿼리로 최고액연봉자와 최저액연봉자의 이름과 연봉
(
	SELECT E.salary AS `연봉`,
	E.`name` AS `사원명`,
	'최고액연봉자' AS `타입`
	FROM emp AS E
	ORDER BY E.salary DESC
	LIMIT 1
)
UNION ALL
(
	SELECT E.salary,
	E.`name`,
	'최저액연봉자' AS `타입`
	FROM emp AS E	
	ORDER BY E.salary ASC
	LIMIT 1
)
ORDER BY `타입` ASC
LIMIT 1;
```

### 문제: 각 부서별 최고연봉자를 출력(부서명, 사원명, 입사일, 연봉)해주세요.

정답
```sql
# 각 부서별 최고연봉자를 출력(부서명, 사원명, 입사일, 연봉)해주세요.
# 1단계 : 각 부서별 최고연봉자의 연봉을 구한다.
SELECT E.deptId AS `부서`,
E.`name` AS `사원명`,
DATE(E.regDate) AS `입사일`,
CONCAT(FORMAT(MAX(E.salary), 0), '만원') AS `최고연봉`
FROM emp AS E
GROUP BY E.deptId;

# 2단계, 전체 사원 중에서 자신이 속한 부서의 최고 연봉과 맞지 않는 사원을 필터링
SELECT E.deptId AS `부서`,
E.`name` AS `사원명`,
DATE(E.regDate) AS `입사일`,
CONCAT(FORMAT(E.salary, 0), '만원') AS `최고연봉`
FROM emp AS E
INNER JOIN dept AS D
ON E.deptId = D.id;

SELECT *
FROM emp AS E
INNER JOIN (
	SELECT E.deptId AS deptId,
	MAX(E.salary) AS maxSalary
	FROM emp AS E
	GROUP BY E.deptId
) AS E2
ON E.deptId = E2.deptId
AND E.salary = E2.maxSalary;

# 3단계 : 입사일 추가, 부서명 추가
SELECT D.`name` AS `부서명`,
E.`name` AS `사원명`,
DATE(E.regDate) AS `입사일`,
CONCAT(FORMAT(E.salary, 0), '만원') AS `최고연봉`
FROM emp AS E
INNER JOIN (
	SELECT E.deptId AS deptId,
	MAX(E.salary) AS maxSalary
	FROM emp AS E
	GROUP BY E.deptId
) AS E2
ON E.deptId = E2.deptId
AND E.salary = E2.maxSalary
INNER JOIN dept AS D
ON E.deptId = D.id;
```

### INDEX, UNIQUE INDEX, SQL_NO_CACHE, EXPLAIN, UUID
```sql
# 데이터베이스 a4가 존재하면 삭제
DROP DATABASE IF EXISTS a4;

# 데이터베이스 a4 생성
CREATE DATABASE a4;

# 데이터베이스 a4 선택
USE a4;

# 회원 테이블 생성, loginId, loginPw, `name`
## 조건 : loginId 칼럼에 UNIQUE INDEX 없이
CREATE TABLE `member` (
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	regDate DATETIME NOT NULL,
	loginId CHAR(50) NOT NULL,
	loginPw VARCHAR(100) NOT NULL,
	`name` CHAR(100) NOT NULL
);

# 회원 2명 생성
## 조건 : (loginId = 'user1', loginPw = 'user1', `name` = '홍길동')
## 조건 : (loginId = 'user2', loginPw = 'user2', `name` = '홍길순')
INSERT INTO `member`
SET regDate = NOW(),
loginId = 'user1',
loginPw = 'user1',
`name` = '홍길동';

INSERT INTO `member`
SET regDate = NOW(),
loginId = 'user2',
loginPw = 'user2',
`name` = '홍길순';

SELECT * FROM `member`;
# 회원 2배 증가 쿼리만들고 회원이 백만명 넘을 때 까지 반복 실행
## 힌트1 : INSERT INTO `tableName` (col1, col2, col3, col4)
## 힌트2 : SELECT NOW(), UUID(), 'pw', '아무개'

SELECT UUID();
SELECT CONCAT(1, 2, 3);
SELECT LENGTH("안녕");

SELECT UUID(), CONCAT(1, 2, 3), LENGTH("안녕");

INSERT INTO `member` (regDate, loginId, loginPw, `name`)
SELECT NOW(), CONCAT('user-', UUID()), CONCAT('user-', UUID()), CONCAT('name-', UUID())
FROM `member`;

# 회원수 확인
SELECT COUNT(*) FROM `member`;

# 검색속도 확인
## 힌트 : SQL_NO_CACHE

SELECT SQL_NO_CACHE *
FROM `member` AS M
WHERE M.loginId = 'user1';

# 유니크 인덱스를 loginID 칼럼에 걸기
## 설명 : mysql이 loginId의 고속검색을 위한 부가데이터를 자동으로 관리(생성/수정/삭제) 한다.
## 설명 : 이게 있고 없고가, 특정 상황에서 어마어마한 성능차이를 가져온다.
## 설명 : 생성된 인덱스의 이름은 기본적으로 칼럼명과 같다.
ALTER TABLE `member` ADD UNIQUE INDEX(loginId);

DESC `member`;

SELECT SQL_NO_CACHE *
FROM `member` AS M
WHERE M.loginId = 'user1';

DESC `member`;

# 검색속도 확인, loginId 가 'user1' 인 회원 검색
SELECT SQL_NO_CACHE *
FROM `member` AS M
WHERE M.loginId = 'user1';

# 인덱스 삭제, `loginId` 이라는 이름의 인덱스 삭제
ALTER TABLE `member` DROP INDEX `loginId`;

DESC `member`;

# 회원 테이블 삭제
DROP TABLE `member`;

# 회원 테이블을 생성하는데, loginId에 uniqueIndex 까지 걸어주세요.
CREATE TABLE `member` (
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	regDate DATETIME NOT NULL,
	loginId CHAR(50) UNIQUE NOT NULL,
	loginPw VARCHAR(100) NOT NULL,
	`name` CHAR(100) NOT NULL
);

# 회원 2명 생성
## 조건 : (loginId = 'user1', loginPw = 'user1', `name` = '홍길동')
## 조건 : (loginId = 'user2', loginPw = 'user2', `name` = '홍길순')
INSERT INTO `member`
SET regDate = NOW(),
loginId = 'user1',
loginPw = 'user1',
`name` = '홍길동';

INSERT INTO `member`
SET regDate = NOW(),
loginId = 'user2',
loginPw = 'user2',
`name` = '홍길순';

SELECT * FROM `member`;

INSERT INTO `member` (regDate, loginId, loginPw, `name`)
SELECT NOW(), CONCAT('user-', UUID()), CONCAT('user-', UUID()), CONCAT('name-', UUID())
FROM `member`;

# 회원수 확인
SELECT COUNT(*) FROM `member`;

SELECT *
FROM `member`
WHERE loginId = 'user1';

# 인덱스 쓰는지 확인
## 힌트 : EXPLAIN SELECT SQL_NO_CACHE * ~
EXPLAIN SELECT SQL_NO_CACHE *
FROM `member` AS M
WHERE M.loginId = 'user1';

EXPLAIN SELECT SQL_NO_CACHE *
FROM `member`
WHERE `name` = 'user1';

SELECT COUNT(*)
FROM `member`;

SET GLOBAL slow_query_log = 'off';
SET GLOBAL long_query_time = 2;
```