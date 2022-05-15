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
- 이건 좀 많이 쓴다.
- on절은 필터링 하는 것이다. 즉, 데이터를 조인하는 대상을 줄이는 것
- 주의
  - 일반조인 : `leftJoin(member.team, team)`
  - on조인 : `from(member).leftJoin(team).on(xxx)`

### 4.10. 조인 - 페치 조인

- 페치 조인은 SQL이 제공하는 기능은 아니고, SQL 조인을 활용하여 연관된 엔티티를 쿼리 한 번에 가져오는 것이다. 주로 성능 최적화에서 많이 사용 (활용편 2편 참고), 페치 조인은 "연관된 엔티티를 한번에 최적화해서 조회하는 기능" 이다.
- 페치 조인은 정말 많이 쓴다.

#### 의문점
1. querydsl이 결국 jpql로 변환된다고 했는데, 그러면 querydsl을 사용하면 jpql은 굳이 사용하는 경우는 없는 건가?

### 4.11. 서브쿼리

- SQL에서도 서브쿼리에서는 Alias가 달라야하기 때문에, querydsl도 Q객체를 생성해준다.

#### from 절의 서브쿼리 한계

- JPA JPQL의 한계점으로 from 절의 서브쿼리 (인라인 뷰)는 지원하지 않는다. -> 따라서, querydsl도 지원하지 않는다. (querydsl이란, 그냥 jpql 빌더 역할을 하는 것일 뿐)
- 해결 방안
  1. 서브쿼리를 조인으로 변경한다. (불가능한 상황이 있을수도 있음)
  2. 애플리케이션에서 쿼리를 2번 분리해서 실행한다.
  3. nativeSQL을 사용한다.

#### SQL에 관해서

- SQL은 데이터를 가져오는 것에 집중! , 화면에서 렌더링되는 데이터들은(예를들면, 날짜 포맷 등등) 화면에서 하는 것이 좋다.
- 어떻게든 화면에 맞춰 데이터를 주려다 보니까, from절 안에 서브쿼리를 넣는 경우가 많다.
- 즉, DB는 정말 데이터를 필터링, 그룹화하여 가져오는데 집중하고 로직들은 모두 애플리케이션이나 화면은 화면 단에서 해주는 것이 좋다.
- 필터링, 그룹화하여 데이터를 최소화하여 가져오는 것이 중요하다.
- 한 방 쿼리에 대해서는 실시간 트래픽이 매우 중요한 애플리케이션 같은 경우에는 쿼리 한번 한번이 중요하지만 대부분의 경우에는 쿼리 한 번으로 복잡하게 짜는 것 보다 차라리 쿼리 2-3번 나누어서 쓰는게 좋을 수가 있다.

### 4.11. Case 문

- select, 조건절(where)에서 사용 가능
- 가급적 DB에서 case를 사용하는 경우는 없도록 한다.
- DB에서는 그냥 DB 안에 있는 데이터를 최소한의 필터링, 그룹화하여 데이터를 줄이는 역할만 하고, CASE 같은 경우는 꼭 필요한 경우가 아닌 경우에는 애플리케이션에서 처리한다.

### 4.12. 상수, 문자 더하기

- Expressions.constant
- concat, stringValue

## 5. 중급 문법

### 5.1. 프로젝션과 결과 반환 - 기본

- 프로젝션 : select 대상 지정
- 프로젝션 대상이 하나면 타입을 명확하게 지정할 수 있음
- 프로젝션 대상이 둘 이상이면 튜플이나 DTO로 조회
- tuple.get()으로 조회
- tuple은 com.querydsl 패키지 안에 존재하기 때문에 이것을 repository 계층에서 사용하는 것은 괜찮지만, service 계층이나 controller 계층에서 사용하는 것은 좋은 설계가 아니다.
  - 나중에, 하부 기술을 querydsl에서 다른 것으로 바꾸더라도 다른 계층에서는 영향도가 없다.
  - 즉, 튜플은 repository 계층 안에서만 사용하고, 바깥으로 나갈때는 dto로 변환해서 내보내자.

### 5.2. 프로젝션과 결과 반환 - DTO 조회

- 순수 JPA에서 DTO로 조회
  - 엔티티로 조회하면, 필요없는 데이터까지 모두 조회해야함
  - 필요한 데이터만 뽑기위해 DTO 활용
  - JPA에선 new -DTO를 제공 (생성자 방식만 지원함) 
- Querydsl은 세 가지 방법으로 DTO를 활용하도록 제공
  - 프로퍼티 접근(setter) : Projections.bean
  - 필드 직접 접근 : Projections.fields (getter/setter 없어도 됨) (private이어도 리플렉션 같은 기술을 활용하여 할 수 있음) - 단, 필드 명이 맞아야 함 -> as()로 해결
  - 생성자 사용 : Projections.constructor, @QueryProjection

### 5.3. 프로젝션과 결과 반환 - @QueryProjection

- 가장 깔끔한 해결 방법이지만, 약간의 단점이 존재
- constructor 방법은 runtime 오류가 나기 때문에 안 좋다. (컴파일 시에 오류 못 잡음)
- @QueryProjection 다 좋지만, 몇 가지 단점이 존재한다.
  1. Q파일을 생성해야함
  2. Dto가 querydsl 기술에 의존성을 갖게 됨

### 5.4. 동적 쿼리 - BooleanBuilder 사용

- 동적 쿼리를 해결하는 두 가지 방식
  1. BooleanBuilder
  2. Where 다중 파라미터 사용
- BooleanBuilder
- Where 다중 파라미터 방법의 장점
  - BooleanBuilder 보다 더 깔끔해짐
  - where문에 null이 들어가면 무시가 된다.
  - 자바 코드를 사용하기 때문에 조립 가능 -> 조립하여 재사용 가능
  - 쿼리의 가독성이 높아진다.