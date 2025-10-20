## Flow의 장점과 SafeCollector

Android Weekly #697 **Understanding the internal of Flow, StateFlow, and SharedFlow**의 Flow에 대해 이해하며 작성하는 글이다.

### Flow의 장점
1. Context preservation
   - Flow를 Collect하는 코루틴 context 내에서만 데이터를 emit하도록 강제한다.
   - withContext와 같은 Context 변경이 이루어지는 경우 런타임에 `IllegalStateException` 에러를 발생시킨다.
   - 이를 통해, 개발자가 의도치 않게 잘못된 스레드에서 emit하는 것을 런타임에서 방지할 수 있다.
2. Exception Transparency
   - emission에 실패할 경우, Flow는 즉시 멍추고 예외를 뿌린다.
   - 데이터를 하나씩 emit하는 Flow의 특성 덕분에, 어느 위치에서 에러가 발생했는지 명확하게 파악이 가능하다.

*Flow에서 Upstream과 Downstream
  - Upstream: 데이터를 emit하는 쪽
  - Downstream: 데이터를 collect하는 쪽

## Flow 빌더와 SafeCollector의 스레드/컨텍스트 검증
<img width="1103" height="242" alt="image" src="https://github.com/user-attachments/assets/e68b0716-e966-44db-add3-58364d36cc80" />  

- `FlowCollector`의 확장 함수 형태로 받은 `block` 람다는 `AbstractFlow`를 오버라이드한 `collectSafely` 함수 내에서 수행된다.  
- `SafeFlow<T>`가 상속하고 있는 `AbstractFlow<T>`는 다음과 같다.
<img width="856" height="721" alt="image" src="https://github.com/user-attachments/assets/ab75c0e4-79d3-47d4-befd-3d6988ef54ea" />

- `collect()`함수는 여기서 final로 오버라이딩되었다. (더 이상 하위 클래스에서 오버라이드 할 수 없음)
- `SafeCollector`는 데이터 emit() 시 현재 코루틴 context를 확인하고, emit한 코루틴과 collect가 실행된 코루틴 context가 일치하지 않을 경우 `IllegalStateException`을 던진다.

## 출처
-🧩 [Understanding the internal of Flow, StateFlow, and SharedFlow](https://www.revenuecat.com/blog/engineering/flow-internals/#the-flow-interface-and-abstractflow-implementation)
