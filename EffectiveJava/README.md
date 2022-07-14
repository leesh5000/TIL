# Effective Java
도서 이펙티브 자바와 백기선님의 강의를 공부하고 정리하는 페이지입니다.

## 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라.

### 생성자의 단점

- 무조건 클래스 명과 같아야 한다.
- **자바의 생성자는 매 번 새로운 인스턴스를 생성한다.**

### 정적 팩터리 메서드의 장점

1. 메서드 명으로 표현할 수 있다.
2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
3. 인터페이스나 하위 타입 객체를 반환할 수 있다.
    1. 유연성을 가질 수 있다.
    2. ServiceLoader를 이용하여 서비스 구현체에 의존적이지 않게 사용이 가능하다.
    3. 자바 8 이후 부터는 인터페이스에 스태틱 메서드를 선언할 수 있다.

### 정적 팩터리 메서드의 단점

1. 정적 팩터리 메서드**만** 제공하는 경우에는 하위 클래스를 만들 수 없다.
2. JavaDoc 문서에 정적 팩터리 메서드는 찾기가 힘들다. (메서드가 많을 경우에)
    1. 네이밍 패턴을 정해서 손쉽게 찾을 수 있도록 한다.

### 핵심 정리

- 정적 팩터리 매서드와 public 생성자는 각각의 장단점이 있으니 이를 이해하고 사용하자. 그렇다 할지라도 정적 팩터리 메서드를 사용하는게 유리한 경우가 더 많다.

## 보충 설명

### 열거 타입

1. 상수 목록을 담을 수 있는 데이터 타입
2. 값을 제한할 수 있다. (Type-Safety 보장 가능)
3. 열거 타입은 JVM 내에 오로지 하나만 만들어진다.
4. enum의 values()를 사용하면 모든 enum 값들을 가져올 수 있다.
5. enum은 생성자, 메서드, 필드를 가질 수 있다.
6. enum은 equals() 보다는 "=="을 사용하여, NPE를 피하도록 하자.

### 플라이웨이트 패턴

1. 객체를 가볍게 만들어 메모리 사용을 줄이기 위한 디자인 패턴

### 인터페이스와 정적 메서드

1. 자바 8과 9의 주요변화
   1. 인터페이스 내부에서 메서드를 정의할 수 있다. (8 버전)
      1. default : 인스턴스에서만 호출할 수 있는 메서드
      2. static : 인스턴스 없이 호출할 수 있는 메서드
   2. static private를 사용할 수 있다.

### 서비스 제공자 프레임워크

### 리플렉션

1. 클래스로더를 통해 읽어온 클래스 정보(거울에 반사된 정보)를 사용하는 기술
2. 리플렉션을 사용해 클래스를 읽어오거나, 인스턴스를 가져오거나 메서드를 실행, 필드 값 조작이 가능하다.

## 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라.

### 아이템 2. 핵심 정리 1 - 생성자 체이닝과 자바빈즈

- 클래스의 필수 값들은 생성자의 매개변수로 받도록 강제하는 것이 좋음
- 생성자 방식
  - 매개변수가 많으면, 어떤 값을 줘야하는지 알기 힘듦
- 자바빈즈 패턴
  - 자바 표준 스펙 중 하나, Getter, Setter 의 네이밍 패턴을 정의한 표준 스펙
  - 장점
    - 객체 생성이 쉬움
  - 단점
    - 필수 값이 세팅이 안된 상태로 객체가 사용될 수도 있음
    - 객체의 일관성이 보장되지 않음
    - 불변 객체로 만들 수 없음

### 아이템 2. 핵심 정리 2 - 빌더

- 자기 자신을 반환함으로써 메서드 체이닝
- 필수 매개변수는 빌더 생성자로 받고, Optional은 따로 받기
- 장점
  - 필드가 많을 때 사용하기 좋음
  - Setter가 없으니 객체를 불변으로 만들 수 있음
- 단점
  - 어려운 코드를 작성해야함
  - 필드가 중복됨
- 롬복
  - 어노테이션 프로세서
    - 어노테이션은 아무런 기능이 없는 그냥 주석 같은 것 임
    - 다만, 어노테이션 프로세스라는 기능이 어노테이션을 컴파일 시점에 읽어들여서 코드를 조작할 수 있는 기능임
  - 어려운 빌더 코드를 작성하지 않아도 됨
  - 단점
    - 다만, 모든 필드를 받는 생성자가 생성되어 노출됨
      - `@AllArgsConstructor(accessLevel=private)`로 막을 수 있음
    - 롬복 빌더의 경우, 필수값을 지정해 줄 수가 없음

### 아이템 2. 핵심 정리 3 - 계층형 빌더

- 부모 빌더에서 self()를 추가하여 하위 타입을 리턴할 수 있도록 함
- self() 매커니즘의 활용도

### 아이템 2. 완벽 공략

### 아이템 2. 완벽 공략 6 - 자바 빈

- 자바 빈이 지켜야할 규약
  - 기본 생성자
    - 기본 생성자는 왜 필요하냐면, 객체를 만들기 편하게 하기 때문이다.
    - 리플렉션으로 인스턴스를 만들려면, 매개변수가 있는 생성자로 만드는 것은 번거로운 일이다.
  - getter, setter
    - 필드에 접근하기 위한 일관적인 방법에 도움을 준다. (자바 빈 스펙)
  - boolean은 isHealthy(getter)
  - serializable 인터페이스를 구현해야함
    - 객체를 저장/조회를 해야하는데, Serializable을 통해서 직/역직렬화를 한다.
  - 이런 규약들을 지켜야 외부에서 자바 빈을 사용할 수 있다.
- 레코드 사용해보기

## 아이템 10. equals는 일반 규약을 지켜 재정의하라

### 아이템 10. 핵심 정리 1 - equals 재정의가 필요없는 경우

- 되도록 `equals()` 재정의를 하지않는 것이 최선이다.
- `equals()` 재정의가 필요하지 않은 경우
  - 싱글톤, Enum 같이 인스턴스가 고유한 경우
  - 인스턴스의 논리적 동치성을 검사할 필요가 없는 경우
    - Object가 제공하는 equals()는 "==" 비교이므로 오직 하나의 인스턴스
    - 대표적으로 문자열
  - 상위 클래스의 equals()가 하위 클래스에도 적절하여 굳이 재정의 할 필요가 없는 경우
    - 대표적으로 AbstractList, AbstractMap
  - 클래스가 private, package-private 이고 equals를 호출할 일이 없는 경우
    - public 클래스의 경우, eqauls()를 명시적으로 호출하지 않다 하더라도 list, set 등에 넣어 사용하면 호출될 수가 있음
- `equals()` 재정의가 필요한 경우
  - 반사성 : A.equals(A) == true
  - 대칭성 : A.equals(B) == B.equals(A)
  - 추이성 : A.equals(B) && B.equals(C), A.equals(C)
  - 일관성 : A.equals(B) == A.equals(B)
  - null-아님 : A.equals(null) == false

### 아이템 10. 핵심 정리 2 - equals 규약 - 반사성, 대칭성

- 대칭성
- 추이성
  - Point 클래스

### 아이템 10. 핵심 정리 3 - equals 규약 - 추이성

- Point 클래스의 추이성 보장하는 방법
  - 상위클래스는 instance of 가 아니고, getClass()로 비교 -> 리스코프 치환 원칙을 위배
- 결론 : 하위 타입에 필드를 추가한 경우, equals() 규약을 만족시킬 방법은 존재하지 않는다.
  - 예) Timestamp, Date

### 아이템 10. 핵심 정리 4 - equals 규약 - 일관성, null 아님

- 그렇다면, 필드가 추가되는 경우에 equals()를 재정의해야한다면 Composition을 사용해서 필드를 추가하자. (상속을 하지 말고)
- 일관성
  - 불변객체는 처음 equals()와 2,3,... 의 equals()는 항상 같아야 한다.
  - 가변객체는 일관성이 위배될 수 있다.
    - 일관성을 보장하기 위해서는 그냥 단순히 메모리에 존재하는 객체만을 사용하여 비교하라.
- null-아님

### 아이템 10. 핵심 정리 5 - equals 구현 방법과 주의 사항

- equals() 구현 방법
  - 객체의 동일성 비교 (자기 자신과 같은지 비교 (==))
  - instance of 로 타입 비교
  - 형 변환
  - 객체의 필드 중 핵심적인 필드들만 같은지 비교
  - 부동소수점(float, double)은 compare로 비교, primitive 는 ==, reference 는 equals()로 비교
  - null도 가능하면, Objects.equals()를 사용
- 하지만, 이런 규약을 다 지키면서 equals()를 구현하기는 어려우니 다음 방법들을 이용하자.
  1. AutoValue 사용
    - 따라야할 규약이 너무 많아 사용하기 힘듦
    - 자바가 의도하는대로 어노테이션 프로세서 사용 (원래의 클래스 파일을 수정하지 않고 새로운 클래스 파일 생성)
  2. lombok 사용
     - 자바가 의도하지 않는대로 어노테이션 프로세서 사용 (원래의 클래스 파일을 수정)
  3. 자바 17 버전을 사용하는 경우에는 자바의 레코드 사용
     - 과제
  4. IDE에서 만들기
     - 필드가 늘어나거나 사라질때마다 매번 다시 만들어야함
     - 까먹을 수도 있음
- 주의사항
  - equals()를 재정의 할 때, 반드시 hashCode도 재정의해야한다.
  - Object가 아닌 타입의 매개변수를 받는 equals() 메서드는 선언해선 안된다.

### 아이템 10. 완벽 공략 24 - Value 기반 클래스

- 값 타입 객체는 불변 객체로 만든다.
  - 객체를 유일하게 식별하는 식별자가 존재해서는 안됨
- 자바17을 사용하는 경우 값 타입 객체를 만들 때, record 활용하는 것을 고려해보자

### 아이템 10. 완벽 공략 25 - StackOverflowError

- stackOverflowError
  - 스택
    - 스택은 한 쓰레드마다 사용하는 공간
    - 스택에는 메서드를 호출할 때마다 스택 프레임이 쌓인다.
    - 스택 프레임 안에는 메서드 호출에 대한 정보들 (매개변수, 메서드에서 참조해야하는 객체를 가르키는 레퍼런스, 돌아갈 위치 등)
  - 힙
    - 힙은 GC가 정리를 해주는 공간
    - 객체의 인스턴스들이 존재하는 공간
  - 큰 스택을 써야하는 경우