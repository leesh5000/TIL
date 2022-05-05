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

## 예제 도메인 모델

### 에제 도메인 모델과 동작확인
- em.flush() : 실제 쿼리를 만들어 DB에 보냄 / em.clear() : 영속성 컨텍스트 초기화
- changeTeam에서 멤버가 중복되는 문제 : 추가 후 제거해줘야 하지만, 이렇게 까지 하는 경우 너무 번거롭다. 일반적으로, changeTeam() 이후에 영속성 컨텍스트를 clear하여 다시 team을 조회하여 중복문제를 해결한다. (team이 연관관계의 주인이 아니기 때문에 가능)
- Test 코드에 @Transactional이 있으면, 테스트가 끝나는 시점에 롤백시킨다.
