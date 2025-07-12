## ViewModel의 StateFlow

아티클을 읽다가 먼저 Flow에 대해 확실히 이해하고 싶어서 공부한 내용을 바탕으로 
오늘은 StateFlow에 대해 이해한만큼 정리해볼까 한다.

### 뷰모델에서의 StateFlow 사용
StateFlow는 UI와 뷰모델간의 데이터를 비동기적으로 전달하는데 주로 사용된다.<br>

**참조하는 객체의 변경을 감지**하고 그 값을 emit(방출)하여 이 StateFlow를 구독한 구독자에게 전달한다. <br>
보통 뷰모델에서 StateFlow는 다음과 같이 선언되고 변경된다.
```
  // 예시
  private val _data = mutableStateFlowOf<List<Int>>(emptyList())
  val data: StateFlow<List<Int>> = _data

  // 변경
  _data.value = newList
```
`_data`는 당연하지만 변경이 가능한 StateFlow이고, `data`는 읽기 전용이다. <br>

기억하고 있어야할 부분은, 새로운 객체가 할당되어야 StateFlow는 변화를 감지한다는 것이다.<br>
즉, 다음과 같은 상황에서는 StateFlow는 변화를 감지하지 못한다.
```
  // 예시
  private val _data = mutableStateFlowOf<MutableList<Int>>(emptyList())
  val data: StateFlow<MutableList<Int>> = _data

  // 변경 MutableList에 값을 추가
  _data.value.add(0)
```
왜냐하면 data가 참조하는 객체의 값은 변화했지만 주소는 변화하지 않았기 때문이다. 
따라서 변화를 전달하고 싶을 때는 새로운 객체를 할당해주어야 한다.<br>

### 이걸 Composable에서는 어떻게 구독할까?
```
  val data = viewModel.data.collectAsState()
```
우리는 Flow가 방출한 데이터를 Collect해서 수집해야한다. <br>

이때, 보통 Composable에서는 `.collect()`보다는 `.collectAsState()` 혹은 `.collectAsStateWithLifeCycle()`를 사용한다.<br>

이유는 Composable의 Recomposition을 위해서다. 데이터가 변화할때마다 UI도 같이 변화해야하는 경우가 많기 때문이다.<br>
State는 값이 변화할때마다 UI를 Recomposition한다.<br>

만약 UI의 변화가 필요하지 않고 값만 처리한다라고 하면, 한 가지 방법으로 `.collect()`와 `LaunchEffect()`를 함께 사용할 수 있다.
```
  LaunchEffect(Unit) {
    viewModel.data.collect { data ->
      // TODO: 값 처리 (UI 변경 없이 로직만 처리할 때)
    }
  }
```
여기서 한 가지 궁금한 점이 들 수 있다. 

### 왜 LaunchEffect 내부에서 Flow를 Collect할까?
`collect()`는 Suspend(중단) 함수다. 따라서 그냥 
``` val data = viewModel.data.collect() ```는 컴파일 에러가 발생한다.<br>
좀 더 자세하게, Flow는 비동기적으로, 시간에 따라 데이터를 방출한다. 방출되는 데이터 간의 Delay가 있을 수도 있다.
Flow는 Collect 관련 함수를 호출할 때 방출을 시작하고, `collect()`는 데이터를 하나씩 받을때까지 기다리기 때문에 Suspend 함수로 정의된다.<br>

따라서 `collect()`를 호출하려면 반드시 코루틴 내부여야 한다. <br>

참고로, `collectAsState()`도 역시 Flow를 수집하지만, 
Jetpack Compose가 자동으로 알아서 Collect를 코루틴 안에서 수행해주기 때문에 따로 코루틴 안에 작성하지 않아도 된다. 

## 참고
- 🤖 ChatGPT (개념을 이해하는 용도로만 활용)
