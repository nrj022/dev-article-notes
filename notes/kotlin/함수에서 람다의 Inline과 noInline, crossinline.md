## 함수에서 람다의 inline과 noinline, crossinline

### inline의 특징
- 메모리 할당 적음 (새로운 람다 객체를 생성하지 않음)
- 추가 호출이 없어 더 나은 퍼포먼스 제공
- 하지만, 파일 크기가 커짐
- non-local return 가능 (람다 내부의 return이 외부 함수를 종료시킬 수 있음)

### 언제 inline을 사용할까?
- 람다의 오버헤드를 제거하고 싶을 때
- non-local return이 필요할 때
- 함수가 작고 자주 사용될 때

### inline 사용을 피해야할 때?
- 함수가 크거나 복잡할 때
- 가끔 호출될 때
- 바이너리 크기를 최적화하고 있을 때

### noinline을 사용하는 경우?
  - 함수 내부에서 일부 람다 객체를 다른 함수의 파라미터로 넘기고 싶을 때
  - inline에서 제외시킬 인자 앞에 noinline 키워드를 붙임

### crossinline이란?
  - inline 함수의 non-local return을 비활성화
  - 다른 스레드, 코루틴에서의 호출이 안전

## 관련 자료
- 🧩 [How to Answer Like a Kotlin Expert: inline, noinline, crossinline, and reified Explained Simply](https://proandroiddev.com/how-to-answer-like-a-kotlin-expert-inline-noinline-crossinline-and-reified-explained-simply-6fd446584fa4)
- 🧩 [[Kotlin] Inline Function 파헤치기](https://velog.io/@haero_kim/Kotlin-Inline-Function-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0)
