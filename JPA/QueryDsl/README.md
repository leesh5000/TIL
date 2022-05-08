# QueryDsl

영한님의 [QueryDsl](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84) 강의를 공부하고 정리하는 페이지입니다.

## 프로젝트 환경설정

- querydsl은 먼저 Q 타입을 뽑아내고, 이것으로 쿼리를 날린다.
- SpringBoot 2.6 이상, Querydsl 5.0 지원 방법 [링크](https://www.inflearn.com/questions/355723)

### 라이브러리 살펴보기
- querydsl-apt는 Q객체를 만들어주기 위한 라이브러리, querydsl-jpa는 실제 쿼리와 관련된 라이브러리

### h2 database 설치

### 스프링 부트 설정 - JPA, DB
- `jpa.properties.hibernate.show_sql`은 하이버네이트 쿼리가 System.out으로 출력이 되고, `logging.level.org.hibernate.SQL`은 logger를 통해 출력
- Test 코드에 @Transactional이 있으면, 기본적으로 모두 롤백을 해버림 -> @Commit 어노테이션을 달아두면 됨
- `logging.level.org.hibernate.type: trace`로 하면 (?)를 볼 수 있음

## 3. 예제 도메인 모델

### 3.1. 에제 도메인 모델과 동작확인
- em.flush() : 실제 쿼리를 만들어 DB에 보냄 / em.clear() : 영속성 컨텍스트 초기화
- changeTeam에서 멤버가 중복되는 문제 : 추가 후 제거해줘야 하지만, 이렇게 까지 하는 경우 너무 번거롭다. 일반적으로, changeTeam() 이후에 영속성 컨텍스트를 clear하여 다시 team을 조회하여 중복문제를 해결한다. (team이 연관관계의 주인이 아니기 때문에 가능)
- Test 코드에 @Transactional이 있으면, 테스트가 끝나는 시점에 롤백시킨다.

## 4. 기본 문법

### 4.1. 시작 - JPQL vs Querydsl
- JPQL은 런타임 시점에 오류 확인 가능, Querydsl은 컴파일 시점에 오류 확인 가능
- JPAQueryFactory는 필드 레벨에서 사용하는 것을 권장 (동시성 문제를 고민할 필요가 없도록 설계되어 있음)

### 4.2. 기본 Q-Type 활용
- static 변수 vs new Q.. 로 생성 -> static 변수를 활용하는 것을 권장 (같은 테이블을 조인 하는 경우에만 new로 선언해서 사용)
- querydsl은 jpql의 빌더 역할을 하는데, 결과적으로 querydsl로 작성한 코드는 jpql이 된다.
- 실행되는 jpql을 보고 싶으면, `spring.jpa.properties.hibernate.use_sql_comments: true` 옵션을 넣으면 된다.

### 4.3. 검색 조건 쿼리
- 많은 검색조건들을 제공 (체이닝)
- and의 경우, 파라미터를 여러 개 넘기면 and가 됨
```java
member.username.eq("member1") // username = 'member1'
member.username.ne("member1") //username != 'member1'
member.username.eq("member1").not() // username != 'member1'
member.username.isNotNull() //이름이 is not null
member.age.in(10, 20) // age in (10,20)
member.age.notIn(10, 20) // age not in (10, 20)
member.age.between(10,30) //between 10, 30
member.age.goe(30) // age >= 30
member.age.gt(30) // age > 30
member.age.loe(30) // age <= 30
member.age.lt(30) // age < 30
member.username.like("member%") //like 검색 
member.username.contains("member") // like ‘%member%’ 검색 
member.username.startsWith("member") //like ‘member%’ 검색
```

### 4.4. 결과 조회
- fetch() : 리스트 조회, 데이터 없으면 빈 리스트 반환 
- fetchOne() : 단 건 조회
  - 결과가 없으면 : null
  - 결과가 둘 이상이면 : com.querydsl.core.NonUniqueResultException 
- fetchFirst() : limit(1).fetchOne()
- fetchResults() : 페이징 정보 포함, total count 쿼리 추가 실행 -> totalCount를 가져와야하기 때문에 쿼리가 2번 나간다.
- fetchCount() : count 쿼리로 변경해서 count 수 조회

### 4.5. 정렬
- `desc(), asc(), nullsLast(), nullsFirst()`

### 4.6. 페이징
- `offset(), limit()`으로 페이징 지원
- 주의! offset은 0부터 시작
- fetchResults는 복잡한 페이징 쿼리의 경우에는 count, contents 쿼리 모두 복잡한 쿼리로 가져오기 때문에 성능을 위해서는 count, contents 쿼리 따로 작성하는 것이 좋다.

### 4.7. 집합
- `count(), sum(), avg(), max(), min(), ...`은 데이터를 조회할 때, 튜플로 조회를 한다. (querydsl이 제공하는 튜플)
- groupBy(), having()도 가능

### 4.8. 조인 - 기본조인
- 세타조인(연관관계가 없이 하는 조인, from절에 두 개 나열, Cartesian 곱)
- 세타조인을 하면, 외부 조인(outer join) 불가능 -> 최신 하이버네이트에서는 join on 절을 사용하면 외부 조인 가능
- join = inner join

### 4.9. 조인 - on절
- on절을 활용한 조인(JPA 2.1부터 지원)
1. 조인 대상 필터링
2. 연관관계 없는 엔티티의 외부 조인

#### 조인 대상 필터링
- (inner)join on == join where (left join인 경우에는 다름)

#### 연관관계 없는 엔티티 외부 조인
- 
