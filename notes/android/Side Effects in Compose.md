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
코루틴 내부에서 실행되는 작업 블록으로, 주로 비동기 작업, 네트워크 혹은 딜레이 등을 처리할 때 주로 사용한다.
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

## 참고 자료
- 🧩 [The simplest way to understand side effects in compose](https://www.droidcon.com/2024/10/28/the-simplest-way-to-understand-side-effects-in-compose/)
