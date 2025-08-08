## Coroutine에서 작업 처리 방식과 다양한 실행 흐름 이해하기

Android Weekly #686 **"Understanding Kotlin Coroutines: A Deep Dive into Blocking vs Non-blocking and Concurrent vs Asynchronous Execution"** 를 이해하며 적는 글이다.

### Coroutine의 Blocking과 Non-Blocking를 비교해보자
Kotlin의 코루틴은 Thread 위에서 동작하며 스레드를 효율적으로 나눠 쓰도록 돕는다.  

이런 **Coroutine의 Blocking은** 무엇을 의미할까? 바로 해당 **코루틴이 스레드를 멈추는 상태(스레드를 점유)** 을 뜻한다.
아래 코드가 Coroutine의 Blocking 예시이다.
```
suspend fun workOnBlockingTask(time: Long) {
    Thread.sleep(time)
    yield()
}
```
여기서 `yield()`는 같은 스레드 내부에 있는 다른 코루틴에게 스레드를 양보하여 실행 기회를 주는 suspend fun의 함수이다. 
이 함수는 스레드를 `time`만큼 중단시킨 후 스레드를 양보한다.  

그렇다면 **Coroutine의 Non-Blocking**은? 바로 **스레드를 멈추지 않고 코루틴만 중단**시켜, 
스레드 내부의 다른 작업이나 코루틴을 실행시킬 수 있는 상태이다.
아래 코드는 그 예시이다.
```
suspend fun workOnNonBlockingTask(time: Long) {
    delay(time)
}
```
suspend fun의 `delay()`함수를 사용하여 코루틴을 일정시간 중단시킨다.  

이 두 Blocking/Non-Blocking 함수를 가지고 다음과 같은 **Test Action**을 작성해볼 수 있다.
```
// blocking work

suspend fun actionA() {
    println("A >> STARTED")
    workOnBlockingTask(500)
    println("A >> Step 1 done")
    workOnBlockingTask(300)
    println("A >> Step 2 done")
    println("A >> DONE")
}

suspend fun actionB() {
    println("B >> Started")
    workOnBlockingTask(200)
    println("B >> Step 1 done")
    workOnBlockingTask(400)
    println("B >> Step 2 done")
    println("B >> DONE")
}

// non-blocking work

suspend fun actionC() {
    println("C >> Started")
    workOnNonBlockingTask(500)
    println("C >> Step 1 done")
    workOnNonBlockingTask(300)
    println("C >> Step 2 done")
    println("C >> DONE")
}

suspend fun actionD() {
    println("D >> Started")
    workOnNonBlockingTask(200)
    println("D >> Step 1 done")
    workOnNonBlockingTask(400)
    println("D >> Step 2 done")
    println("D >> DONE")
}
```

### Sequential/Concurrent/Parallelism의 차이 이해하기

**+ 내용 추가 예정**

## 출처
- 🧩 [Understanding Kotlin Coroutines: A Deep Dive into Blocking vs Non-blocking and Concurrent vs Asynchronous Execution](https://proandroiddev.com/understanding-kotlin-coroutines-a-deep-dive-into-blocking-vs-non-blocking-and-concurrent-vs-7667dfe77fbb)
