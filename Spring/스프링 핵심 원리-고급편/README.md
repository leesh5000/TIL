# 스프링 핵심 원리 - 고급편
인프런 김영한님의 [스프링 핵심 원리 - 고급편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8) 강의를 공부하고 정리하는 페이지입니다.

[강의 소스코드](https://github.com/leesh5000/Spring-Practice/tree/master/%EC%8A%A4%ED%94%84%EB%A7%81%20%ED%95%B5%EC%8B%AC%20%EC%9B%90%EB%A6%AC%20%EA%B3%A0%EA%B8%89%ED%8E%B8/advanced)는 유료 강의이기 때문에 Private 저장소에 저장합니다.

## 1. 예제 만들기

- 요구사항은 모두 만족했지만, TraceId를 매소드의 파라미터로 넘기는 것은 변경 사항에 유연하게 대응할 수 없다. -> 다른 대안이 필요


## 2. ThreadLocal

- traceIdHolder 필드를 사용하는 것은 동시성 문제를 발생시킬 수 있다. (fieldLogTrace가 싱글톤으로 등록되었기 때문에)
- `2022-05-03 23:41:22.071  INFO 96432 --- [nio-8080-exec-5] h.advanced.trace.logtrace.FieldLogTrace  : [62ad053b] |-->OrderServiceV1.orderItem()` 톰캣에서 제공하는 thread ID `[nio-8080-exec-5]`

### 2.4. 동시성 문제 - 예제 코드
- 공유 자원에 대한 동시 접근으로 생기는 문제에 대해 알아봅시다.
- 싱글톤 객체의 필드에 접근할 떄 주의
- 지역변수에서는 동시성 문제가 발생하지 않는다. 왜냐하면, 쓰레드 마다 지역변수에서는 각가 다른 메모리 영역이 할당되기 때문이다.
- 싱글톤 객체의 멤버변수, static과 같은 공용 필드와 같은 곳에서 동시성 문제가 발생한다.

따라서, 이와같이 싱글톤 객체의 필드를 사용하면서 동시성 문제를 해결하기 위해서 사용하는 것이 `ThreadLocal`이다.
