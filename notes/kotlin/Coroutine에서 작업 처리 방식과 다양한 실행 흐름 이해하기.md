## Coroutine에서 작업 처리 방식과 다양한 실행 흐름 이해하기

Android Weekly #686 **"Understanding Kotlin Coroutines: A Deep Dive into Blocking vs Non-blocking and Concurrent vs Asynchronous Execution"** 를 이해하며 적는 글이다.

### Coroutine의 Blocking과 Non-Blocking를 비교해보자
Kotlin의 코루틴은 Thread 위에서 동작하며 스레드를 효율적으로 나눠 쓰도록 돕는다.  

이런 **Coroutine의 Blocking은** 무엇을 의미할까? 바로 해당 **코루틴이 스레드를 멈추는 상태(스레드를 점유함)** 를 뜻한다.
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

### Sequential/Concurrent/Parallelism 이해하기
1. Sequential Execution (순차 실행)
   
   아래와 같이 Blocking / Non-Blocking 함수를 `runBlocking`을 사용해 수행하면, 하나의 코루틴 안에서 코드가 동기 실행된다. 따라서 예상한대로 각각의 Action은 순차적으로 실행되며, 수행 시간은 둘 다 약 1440 ms이다.
    ```
    // blocking work
    val blockingTime = measureTimeMillis {
        runBlocking {
            actionA()
            actionB()
        }
    }
    println("Blocking operations: $blockingTime ms")
    
    // non-blocking work
    val nonBlockingTime = measureTimeMillis {
        runBlocking {
            actionC()
            actionD()
        }
    }
    println("Non-blocking operations: $nonBlockingTime ms")
    ```
2. Concurrent Excution (동시 실행)
   
    이번엔 `launch`를 사용하여 하나의 runBlocking 코루틴 안에 각각 코루틴을 생성해 actionA와 B를 동시 실행해본다.
    
    이런 경우, **blocking work**는 **`workOnBlockingTask`의 `yield()`가 효과를 내며 actionA와 B가 번갈아 실행**된다. 하지만 둘 다 하나의 Thread에서 실행되므로 **실행시간은 스레드의 `sleep()`시간이 영향을 주며** 약 1400 ms 이다.  
    
    **non-blocking work**는 어떨까? **`workOnNonBlockingTask`의 `delay()`는 코루틴을 지연**시키므로 actionC의 코루틴이 지연시 actionD의 코루틴이 수행되고 그 반대로도 마찬가지다. 따라서 B의 실행시간은 약 800 ms으로 **순차 실행보다 빠르다.**  
    ```
    // blocking work
    val blockingTime = measureTimeMillis {
        runBlocking {
            launch { actionA() }
            launch { actionB() }
        }
    }
    println("Blocking operations: $blockingTime ms")
    
    // non-blocking work
    val nonBlockingTime = measureTimeMillis {
        runBlocking {
            launch { actionC() }
            launch { actionD() }
        }
    }
    println("Non-blocking operations: $nonBlockingTime ms")
    ```
3. Parallelize Execution (병렬 실행)

   병렬 실행은 Thread를 사용해 수행한다. `runBlocking(Dispatchers.Default)`은 **`runBlocking`이 메인 스레드 대신 `Dispatchers.Default`의 스레드 풀에서 코루틴을 시작**한다. 스레드 풀에 유휴 스레드가 있다면 **actionA, B의 코루틴이 각각 다른 스레드에서 동시 실행되어 병렬 실행이 가능**하다.

   여기서 2번의 동시 실행과 3번의 병렬 실행의 차이를 정리하자면, 그 차이는 **정말 작업이 동시에 실행되는지**에 있다. **동시 실행은** 하나의 스레드에서 작업을 쪼개어 번갈아 실행하면서 **작업이 동시에 수행되는 것 처럼 보이**는 반면, **병렬 실행은 서로 다른 스레드에서 정말로 "동시"에 작업이 수행**된다.

   따라서 아래 코드에서 **blocking work, non-blocking work은 모두 각각 action의 코루틴이 서로 다른 스레드에서 병렬 실행**되며 실행시간이 약 820 ms가 된다.

    ```
    // blocking work
    val blockingTime = measureTimeMillis {
        runBlocking(Dispatchers.Default) {
            launch { actionA() }
            launch { actionB() }
        }
    }
    println("Blocking operations: $blockingTime ms")
    
    // non-blocking work
    val nonBlockingTime = measureTimeMillis {
        runBlocking(Dispatchers.Default) {
            launch { actionC() }
            launch { actionD() }
        }
    }
    println("Non-blocking operations: $nonBlockingTime ms")
    ```
   
## 출처
- 🧩 [Understanding Kotlin Coroutines: A Deep Dive into Blocking vs Non-blocking and Concurrent vs Asynchronous Execution](https://proandroiddev.com/understanding-kotlin-coroutines-a-deep-dive-into-blocking-vs-non-blocking-and-concurrent-vs-7667dfe77fbb)
