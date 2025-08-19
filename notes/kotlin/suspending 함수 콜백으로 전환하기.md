## suspending 함수 콜백으로 전환하기

Android Weekly #688 **"Suspending functions or flows into callbacks"** 를 읽고 작성하는 글이다.

### 콜백 ?
먼저, 콜백이란? 어떤 동작이 끝났을 때 호출되는 함수다.  

안드로이드에서 서버 호출, 버튼 클릭 등은 비동기적으로 처리된다. 즉, 백그라운드에서 서버 응답이 올 때까지 기다리거나, 사용자가 버튼을 클릭할 때까지 기다린다는 것이다. 그리고 해당 이벤트가 발생하면 미리 전달해 놓은 콜백 함수를 호출하여 해당 이벤트를 처리한다.
```
button.setOnClickListener {
    println("버튼 클릭됨")
}
```

### Flow ?
콜백과의 차이점 측면에서 Flow를 바라보자. 사용자가 1초 동안 버튼을 5번 클릭했다고 가정하자. 

- **콜백에 경우에는?**  

  5번의 콜백 호출이 발생한다. 동일한 동작이 1초간 5번 반복될 것이다. 이를 방지하고 싶다면, 한 번 클릭되었을 때 몇 초간 버튼을 비활성화하는 등 추가적인 구현이 필요하다.

- **Flow를 사용한다면 ?**  
  
  버튼 클릭 이벤트를 Flow로 처리한다는 것은 클릭 이벤트가 스트림처럼 전달된다는 것이다. 그렇다면 1초동안 5번 수행한 버튼 클릭 이벤트를 연속적으로 collect 할 수 있다.
  ```
  val clicksFlow = callbackFlow<Unit> {
    button.setOnClickListener { trySend(Unit) }
    awaitClose { button.setOnClickListener(null) }
  }
  
  lifecycleScope.launch {
      clicksFlow
          .debounce(1000) // 연속 클릭 필터링
          .collect {
              println("버튼 클릭됨")
          }
  }
  ```
  
  만약 1초에 한 번으로 이벤트를 처리를 제한하고 싶다면, `debounce(1000)`와 같은 연산자를 사용하면 쉽게 구현이 가능하다. 이 함수는 1000ms 동안 발생한 이벤트 중 마지막 이벤트만 전달한다. 

### Flow -> 콜백 전환
이벤트를 flow로 처리한 경우, 
