## `Result<T>` Class와 RunCatching

`Result<T>`는 `T`타입의 **성공 결과** 또는 예외를 포함한 **실패 결과**를 **캡슐화**하는 도구로, 
이를 활용하면 예외를 값처럼 다룰 수 있어 코드의 흐름을 더 명확하게 유지할 수 있다.<br>
<br>
이와 함께 사용하는 `RunCatching` 함수는, 함수 실행 중에 발생하는 예외 상황을 자동으로 캐치하고, 
**결과값을 `Result`오브젝트로 랩핑**해 반환함으로써 예외 처리와 반환값을 하나의 흐름으로 통합할 수 있게 해준다.
<br><br>

## `Result` 사용의 주요 장점

1. **Explicit Error Types**<br>
   : 실패 가능성이 있는 함수가 명시적으로 드러남
2. **Composition**<br>
   : `map`, `flatMap` 등을 사용해 실패 가능성이 있는 연산들을 쉽게 연결 가능
3. **Deferred Error Handling**<br>
   : 오류 처리 로직을 나중으로 분리 가능
4. **Predictable Control Flow**<br>
   : 일반적인 `try-catch`방식이 예외 발생 시 흐름을 "점프"하는 것과 달리 `Result`는 값처럼 다룰 수 있어 흐름이 끊기지 않고 명확함
<br>

## `Result` 클래스와 함께 사용하는 함수

| 함수 이름             | 람다 실행 조건  | 예외 처리               | 반환 타입       | 주요 용도                        |
| ----------------- | --------- | ------------------- | ----------- | ---------------------------- |
| `map`             | 성공일 때만 실행 | ❌ 예외 발생 시 앱 크래시     | `Result<R>` | 단순한 값 변환                     |
| `mapCatching`     | 성공일 때만 실행 | ✅ 예외를 `Failure`로 처리 | `Result<R>` | 예외 발생 가능성 있는 값 변환            |
| `flatMap`         | 성공일 때만 실행 | ✅ `Result` 유지       | `Result<R>` | `Result`를 반환하는 함수 연결 (중첩 제거) |
| `recover`         | 실패일 때만 실행 | ❌ 예외 발생 시 앱 크래시     | `Result<T>` | 실패를 기본값으로 대체                 |
| `recoverCatching` | 실패일 때만 실행 | ✅ 예외를 `Failure`로 처리 | `Result<T>` | 실패를 예외 안전하게 대체               |
<br>

## `Result` 사용을 위한 모범사례

1. **일관성 있게 사용하기**<br>
   `Result`와 `try-catch` 중 하나를 골라 코드 전반에 걸쳐 일관성 있게 선택
2. **어떤 오류가 반환될 수 있는지 문서화하기**<br>
   함수가 어떤 오류를 실패로 반환할 수 있는지, 그 이유와 조건을 주석이나 문서로 설명
3. **`Result`와 예외 방식을 섞어 쓰지 않기**<br>
   한 함수 안에서 `Result`와 `try-catch`를 섞어 쓰지 않을 것
4. **의미 있는 변환 함수 사용하기 (`map`, `flatMap`, `recover` 등)**<br>
   변환 함수를 사용하여 깔끔하고 의미 있는 코드로 작성
5. **성공과 실패 모두 처리하기**<br>
   항상 `Success`와 `Failure` 두 경우를 모두 처리할 것
6. **중첩된 `runCatching` 피하기**<br>
   `runCatching { runCatching { ... } }`처럼 중첩된 형태를 피하고, `flatMap`으로 연결
8. **성능이 중요한 곳에서는 신중하게 사용하기**<br>
   `Result`는 내부적으로 객체를 생성하므로, 성능이 민감한 영역에서는 주의할 것 (반복문 내부, 실시간 처리 루틴 등)
9. **성공/실패 상황 모두 테스트하기**<br>
   `Result`는 실패도 값으로 다루기 때문에, 테스트 코드에서 성공과 실패 케이스를 모두 검증할 것
<br>

## 참고 자료
- 🧩 [Elegant Error Handling in Kotlin: Using runCatching and Result](https://carrion.dev/en/posts/runcatching-result-pattern/#transforming-and-chaining-results)
   
