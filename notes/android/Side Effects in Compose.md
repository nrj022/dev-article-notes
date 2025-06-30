## Side Effects in Compose
선언형 UI 방식인 Jetpack Compose에서 데이터베이스 업데이트, 로그 기록, 네트워크 요청 등의 UI 계층 밖에서 일어나는 변화나 작업을 처리하기 위해 사용하는 메커니즘
<br><br>
예를 들어, API 호출이 필요할 때 컴포저블 함수 내에서 그냥 코드를 작성하면 API 호출은 UI가 Recomposition(재구성)될 때마다 매번 실행된다. 이런 경우 Side Effect를 사용하면, UI 계층과 분리되어 통제된 방식으로 작업을 수행할 수 있다.

## Key Side Effect APIs in Jetpack Compose
### SideEffect
Composable 함수가 re-composed(재구성)될 때마다 호출되는 작업 블록으로, 
Composable 함수가 렌더링을 끝낸 후에 1번 실행됨을 보장한다.<br>
<pre><code>@Composable
fun ExampleSideEffect(name: String) {
    Text("Hello, $name")
    SideEffect {
        Log.d("ExampleSideEffect", "Composed with name: $name")
    }
}</code></pre>

### LaunchedEffect
**코루틴 내부에서 실행**되는 작업 블록으로, 주로 비동기 작업, 네트워크 혹은 딜레이 등을 처리할 때 주로 사용한다.
특정 Key의 변화 따라 작업을 실행시키기 위해 사용되며 
처음 Composable이 수행될 때, 그리고 Key가 변화할 때 작동한다. 
또한, 키가 변화하면 이전에 이 블럭이 실행했던 코루틴을 취소한다.

<pre><code>@Composable
fun ExampleLaunchedEffect(userId: String) {
    var userName by remember { mutableStateOf("") }

    LaunchedEffect(userId) {
        // Fetch user data when userId changes.
        userName = fetchUserName(userId)
    }

    Text("User name is $userName")
}</code></pre>

### rememberUpdatedState
코루틴에서 최신 상태를 사용하고 싶을 때 사용하며, 가장 최신 값이 작업 블록에서 사용되도록 보장한다.
<pre><code>
  @Composable
fun ExampleRememberUpdatedState(onTimeout: () -> Unit) {
    val currentOnTimeout by rememberUpdatedState(onTimeout)

    LaunchedEffect(Unit) {
        delay(5000)
        currentOnTimeout() // Uses the latest version of onTimeout.
    }
}</code></pre>

### DisposableEffect
**리소스 정리나 리스너 등록, 해제 같은 non-suspend 정리**에 적합하고 **코루틴을 실행하지 않는다.** <br><br>
화면에 해당 UI가 나타나면 내부에 정의된 리소스가 등록되고 더 이상 화면에 필요하지 않는 UI를 Compose가 컴포지션에서 제거할 때 DisposableEffect 내부의 `onDispose` 함수가 실행된다. <br><br>
Compose는 선언형이기 때문에 직접적으로 lifecycle의 onDestory 같은 것을 사용하지 않는다. 그래서 어떤 리소스 정리를 하고 싶다면, 컴포저블이 사라질 때(cleanup) 이를 감지할 방법이 필요하다.
<pre><code>
    @Composable
fun ExampleDisposableEffect() {
    DisposableEffect(Unit) {
        val listener = MyListener()
        listener.startListening()

        onDispose {
            listener.stopListening() // 컴포저블이 컴포지션을 떠날 때 Clean up 수행
        }
    }

    Text("Listening for updates...")
}
</code></pre>

### productState
**Compose state가 아닌 값을 State 값으로 변환**하여 사용할 수 있다. **코루틴을 실행**하고 상태값을 관찰하며 계속해서 업데이트한다.
<pre><code>
    @Composable
fun Example(userId: String) {
    val userNameState: State<String> = produceState(initialValue = "Loading...", userId) {
        val result = fetchUserNameFromNetwork(userId)
        value = result  // 상태 업데이트
    }

    Text(text = userNameState.value)  // 여기서 최신 값을 읽음
}
</code></pre>

## snapshotFlow
상태 변화를 Flow(비동기 스트림)로 만들어서 연속적인 변화를 다룰 수 있다. `collect`함수를 사용하면 변화하는 값을 실시간으로 받아 처리할 수 있다. 또한 `filter`, `map` 등의 연산자도 사용이 가능하다. <br><br>
`LaunchedEffect`로 상태 변화를 감지하면 상태가 변화할 때마다 기존 코루틴이 취소되고 새로 실행되어 상태가 자주 변화하는 경우 불필요한 오버헤드가 생길 수 있지만, snapshotFlow는 하나의 코루틴 안에서 변화만 관찰하여 상태 변화가 잦은 경우 더 효율적이다.
<pre><code>
// 상태 변화가 잦고, 변화값을 연속으로 처리할 때
val query by remember { mutableStateOf("") }

LaunchedEffect(Unit) {
    snapshotFlow { query }
        .debounce(300) // 300ms 동안 입력 멈추면
        .collect { searchText ->
            performSearch(searchText) // 검색 API 호출
        }
}
</code></pre>
<pre><code>
// 상태 변화를 여러 단계로 필터링, 변환해서 처리할 때
val temperature by remember { mutableStateOf(0) }

LaunchedEffect(Unit) {
    snapshotFlow { temperature }
        .filter { it > 30 }  // 30도 이상일 때만
        .collect {
            showWarning("It's hot!")
        }
}
</code></pre>
| 상황                        | LaunchedEffect      | snapshotFlow         |
| ------------------------- | ------------------- | -------------------- |
| 상태 변경 시 작업 실행             | O                   | O                    |
| 상태 자주 바뀌면 리소스 낭비          | **가능성 있음 (재시작 반복)** | **없음 (코루틴 1개 유지)**   |
| 연산자 활용 (filter, debounce) | X                   | ✅ 다양한 연산자 가능         |
| 반응 방식                     | 상태 변경 시 코루틴 재실행     | 상태 변경을 Flow로 수집해서 처리 |

### rememberCoroutineScope (+ viewModelScope)
- Scope: 코루틴이 언제 시작되고 언제 취소될지 결정하는 작업의 생명 주기 범위
- `remember`의 역할: 재컴포지션이 발생해도 동일한 Scope를 재사용하여 불필요한 객체 생성 방지
- `remeberCoroutineScope()`: Composable의 생명 주기에 맞춰 자동으로 취소되는 CoroutineScope를 만들어 기억하는 함수 (*그냥 `CoroutineScope()`는 일반 클래스로 컴포저블의 상태를 알 수 없다)
- `viewModelScope`: 뷰모델의 생명 주기를 감지하는 CoroutineScope (ViewModel은 재컴포지션이 발생하지 않으므로 Scope를 기억할 필요없다)


## 참고 자료
- 🧩 [The simplest way to understand side effects in compose](https://www.droidcon.com/2024/10/28/the-simplest-way-to-understand-side-effects-in-compose/)
