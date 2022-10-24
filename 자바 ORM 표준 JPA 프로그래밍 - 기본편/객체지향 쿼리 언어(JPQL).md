# 객체지향 쿼리 언어(JPQL)

## 객체지향 쿼리 언어 소개

### JPA는 다양한 쿼리 방법을 지원

- JPQL
    - 가장 단순한 조회 방법이다
        - EntityManager.find()
        - 객체 그래프 탐색(a.getB().getC())
    - JPA를 사용하면 엔티티 객체를 중심으로 개발한다
    - 검색을 할 때도 **테이블이 아닌 엔티티 객체를 대상으로 검색한다**
    
    ```java
    //검색
     String jpql = "select m From Member m where m.name like ‘%hello%'"; 
     List<Member> result = em.createQuery(jpql, Member.class).getResultList()
    ```
    
    - 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리이다
- JPA Criteria
    - 문자가 아닌 자바코드로 JPQL을 작성할 수 있다
    - JPQL 빌더 역할이다
    - JPA 공식 기능이다
    - **단점: 너무 복잡하고 실용성이 없다.**
    - Criteria 대신에 **QueryDSL 사용 권장한다**
- QueryDSL
    
    ```java
    //JPQL 
     //select m from Member m where m.age > 18
    JPAFactoryQuery query = new JPAQueryFactory(em);
     QMember m = QMember.member; 
    
     List<Member> list = 
    	  query.selectFrom(m)
    		.where(m.age.gt(18)) 
    		.orderBy(m.name.desc())
    		.fetch();
    ```
    
    - 문자가 아닌 자바코드로 JPQL을 작성할 수 있다
    - JPQL 빌더 역할이다
    - 컴파일 시점에 문법 오류를 찾을 수 있다
    - 동적쿼리 작성 편리하다
    - 단순하고 쉽다
    - 실무 사용 권장한다
- 네이티브 SQL
    - JPA가 제공하는 SQL을 직접 사용하는 기능이다
    - JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능이다
    - 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트다
    
    ```java
    String sql = 
     “SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = ‘kim’"; 
    List<Member> resultList = 
     em.createNativeQuery(sql, Member.class).getResultList();
    ```
    
- JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용한다
    - JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스등을 함께 사용 가능하다
    - 단 영속성 컨텍스트를 적절한 시점에 강제로 플러시 필요하다
    - 예) JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트
    수동 플러시이다

## JPQL - 기본 문법과 기능

### JPQL 소개

- JPQL은 객체지향 쿼리 언어다.따라서 테이블을 대상으로 쿼리 하는 것이 아니라 엔티티 객체를 대상으로 쿼리로한다.
- JPQL은 SQL을 추상화해서 특정데이터베이스 SQL에 의존하지 않는다.
- JPQL은 결국 SQL로 변환된다.

### JPQL 문법

```sql
select_문 :: = 
 select_절
 from_절
 [where_절] 
 [groupby_절] 
 [having_절] 
 [orderby_절] 
update_문 :: = update_절 [where_절] 
delete_문 :: = delete_절 [where_절]
```

- select m from Member as m where m.age > 18
- 엔티티와 속성은 대소문자 구분O (Member, age)
- JPQL 키워드는 대소문자 구분X (SELECT, FROM, where)
- 엔티티 이름 사용, 테이블 이름이 아님(Member)
- 별칭은 필수(m) (as는 생략가능)

### TypeQuery, Query

- TypeQuery: 반환 타입이 명확할 때 사용한다
    
    ```java
    TypedQuery<Member> query = 
     em.createQuery("SELECT m FROM Member m", Member.class);
    ```
    
- Query: 반환 타입이 명확하지 않을 때 사용한다
    
    ```java
    Query query = 
     em.createQuery("SELECT m.username, m.age from Member m");
    ```
    

### 결과 조회 API

- query.getResultList(): **결과가 하나 이상일 때**, 리스트 반환한다
    - 결과가 없으면 빈 리스트 반환한다
- query.getSingleResult(): **결과가 정확히 하나**, 단일 객체 반환한다
    - 결과가 없으면: javax.persistence.NoResultException
    - 둘 이상이면: javax.persistence.NonUniqueResultException

## 프로젝션

- SELECT 절에 조회할 대상을 지정하는 것이다
- 프로젝션 대상: 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자등 기본 데이터 타입)이다
- SELECT m FROM Member m -> 엔티티 프로젝션
- SELECT m.team FROM Member m -> 엔티티 프로젝션
- SELECT m.address FROM Member m -> 임베디드 타입 프로젝션
- SELECT m.username, m.age FROM Member m -> 스칼라 타입 프로젝션
- DISTINCT로 중복 제거 한다

### 프로젝션 - 여러 값 조회

- SELECT m.username, m.age FROM Member m
- 1. Query 타입으로 조회한다
- 2. Object[] 타입으로 조회한다
- 3. new 명령어로 조회한다
    - 단순 값을 DTO로 바로 조회한다
    SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m
    - 패키지 명을 포함한 전체 클래스 명 입력한다
    - 순서와 타입이 일치하는 생성자 필요하다

## 페이징 API

- JPA는 페이징을 다음 두 API로 추상화 ㅎ나다
- **setFirstResult**(int startPosition) : 조회 시작 위치 (0부터 시작)
- **setMaxResults**(int maxResult) : 조회할 데이터 수
- 페이징 API예시

```java
//페이징 쿼리
 String jpql = "select m from Member m order by m.name desc";
 List<Member> resultList = em.createQuery(jpql, Member.class)
 .setFirstResult(10)
 .setMaxResults(20)
 .getResultList();
```

- 페이징 API - MySQL 방언

```sql
SELECT
 M.ID AS ID,
 M.AGE AS AGE,
 M.TEAM_ID AS TEAM_ID,
 M.NAME AS NAME 
FROM
 MEMBER M 
ORDER BY
 M.NAME DESC LIMIT ?,?
```

- 페이징 API - Oracle 방언

```sql
SELECT * FROM
 ( SELECT ROW_.*, ROWNUM ROWNUM_ 
 FROM
 ( SELECT
 M.ID AS ID,
 M.AGE AS AGE,
 M.TEAM_ID AS TEAM_ID,
 M.NAME AS NAME 
 FROM MEMBER M 
 ORDER BY M.NAME 
 ) ROW_ 
 WHERE ROWNUM <= ?
 ) 
WHERE ROWNUM_ > ? 
```

## 조인

- 내부 조인:
    - SELECT m FROM Member m [INNER] JOIN m.team t
- 외부 조인:
    - SELECT m FROM Member m LEFT [OUTER] JOIN m.team t
- 세타 조인:
    - select count(m) from Member m, Team t where m.username = [t.name](http://t.name/)

### 조인 - ON 절

- 조인 대상 필터링
- 예) 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인
    - JPQL:
        - SELECT m, t FROM Member m LEFT JOIN m.team t on [t.name](http://t.name/) = 'A'
    - SQL:
        - SELECT m.*, t.* FROM
        Member m LEFT JOIN Team t ON m.TEAM_ID=[t.id](http://t.id/) and t.name='A'
- 연관관계 없는 엔티티 외부 조인(하이버네이트 5.1부터)
- 예) 회원의 이름과 팀의 이름이 같은 대상 외부 조인
- JPQL:
    - SELECT m, t FROM
    Member m LEFT JOIN Team t on m.username = [t.name](http://t.name/)
- SQL:
    - SELECT m.*, t.* FROM
    Member m LEFT JOIN Team t ON m.username = [t.name](http://t.name/)

## 서브 쿼리

- 나이가 평균보다 많은 회원
    - select m from Member m
    where m.age > (select avg(m2.age) from Member m2)
- 한 건이라도 주문한 고객
    - select m from Member m
    where (select count(o) from Order o where m = o.member) > 0

### 서브 쿼리 지원 함수

- [NOT] EXISTS (subquery): 서브쿼리에 결과가 존재하면 참
    - 예제
        - 팀A 소속인 회원
        select m from Member m
        where exists (select t from m.team t where [t.name](http://t.name/) = ‘팀A')
    - {ALL | ANY | SOME} (subquery)
    - ALL 모두 만족하면 참
        - 예제
            - 전체 상품 각각의 재고보다 주문량이 많은 주문들
            select o from Order o
            where o.orderAmount > ALL (select p.stockAmount from Product p)
    - ANY, SOME: 같은 의미, 조건을 하나라도 만족하면 참
        - 예제
            - 어떤 팀이든 팀에 소속된 회원
            select m from Member m
            where m.team = ANY (select t from Team t)
- [NOT] IN (subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

### JPA 서브 쿼리 한계

- JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용 가능하다
- SELECT 절도 가능(하이버네이트에서 지원)하다
- **FROM 절의 서브 쿼리는 현재 JPQL에서 불가능하다**
    - **조인으로 풀 수 있으면 풀어서 해결한다**

### JPQL 타입 표현

- 문자: ‘HELLO’, ‘She’’s’
- 숫자: 10L(Long), 10D(Double), 10F(Float)
- Boolean: TRUE, FALSE
- ENUM: jpabook.MemberType.Admin (패키지명 포함)
- 엔티티 타입: TYPE(m) = Member (상속 관계에서 사용)
- SQL과 문법이 같은 식
- EXISTS, IN
- AND, OR, NOT
- =, >, >=, <, <=, <>
- BETWEEN, LIKE, IS NULL

## 조건식 - CASE 식

### 기본 CASE 식

```sql
select
	 case when m.age <= 10 then '학생요금'
				when m.age >= 60 then '경로요금'
				else '일반요금'
    end
from Member m
```

### 단순 CASE 식

```sql
 
select
	case [t.name](http://t.name/)
			when '팀A' then '인센티브110%'
			when '팀B' then '인센티브120%'
			else '인센티브105%'
	end
from Team t
```

- COALESCE: 하나씩 조회해서 null이 아니면 반환한다

```sql
select coalesce(m.username,'이름 없는 회원') from Member m
```

- NULLIF: 두 값이 같으면 null 반환, 다르면 첫번째 값 반환한다

```sql
select NULLIF(m.username, '관리자') from Member m
```

### JPQL 기본 함수

- CONCAT
- SUBSTRING
- TRIM
- LOWER, UPPER
- LENGTH
- LOCATE
- ABS, SQRT, MOD
- SIZE, INDEX(JPA 용도)

### 사용자 정의 함수 호출

- 하이버네이트는 사용전 방언에 추가해야 한다.
- 사용하는 DB 방언을 상속받고, 사용자 정의 함수를 등록한다.
    
    `select function('group_concat', [i.name](http://i.name/)) from Item i`
