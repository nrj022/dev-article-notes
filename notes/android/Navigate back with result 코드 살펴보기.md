## Navigate back with result 코드 살펴보기

Android Weekly #693 **Navigate back with results in Jetpack Compose Navigation**의 코드를 살펴보고자 한다.

### Navigate back with result?
Compose에서 Activity에서 예전에 제공하던 startActivityForResult() 과 같이, 화면을 이동해서 수행한 데이터를 다시 이전 화면으로 전달하는 기능을 구현한 것이다.

작성자가 제공한 코드는 다음과 같다. 
```
private const val RESULT_KEY = "result"

fun <T> NavController.navigateBackWithResult(value: T) {
    previousBackStackEntry?.savedStateHandle?.set(RESULT_KEY, value)
    navigateUp()
}

inline fun <reified T> NavController.navigateBackWithSerializableResult(value: T) {
    navigateBackWithResult(Json.encodeToString(value))
}

suspend fun <T> NavController.navigateForResult(route: Any): T? =
    suspendCancellableCoroutine { continuation ->
        val currentNavEntry = currentBackStackEntry
            ?: throw IllegalStateException("No current back stack entry found")

        navigate(route)

        val lifecycleObserver = object : LifecycleEventObserver {
            override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
                if (event == Lifecycle.Event.ON_START) {
                    continuation.resume(currentNavEntry.savedStateHandle[RESULT_KEY])
                    currentNavEntry.savedStateHandle.remove<T>(RESULT_KEY)
                    currentNavEntry.lifecycle.removeObserver(this)
                }
            }
        }

        currentNavEntry.lifecycle.addObserver(lifecycleObserver)

        continuation.invokeOnCancellation {
            currentNavEntry.savedStateHandle.remove<T>(RESULT_KEY)
            currentNavEntry.lifecycle.removeObserver(lifecycleObserver)
        }
    }

suspend inline fun <reified T> NavController.navigateForSerializableResult(route: Any): T? {
    val result: String = navigateForResult(route) ?: return null
    return Json.decodeFromString(result)
}
```

하나씩 살펴보면 다음과 같다.  

**1. `navigateBackWithResult`: 이전 화면에 데이터 저장 후 이전 화면으로 돌아가기**
```
fun <T> NavController.navigateBackWithResult(value: T) {
    previousBackStackEntry?.savedStateHandle?.set(RESULT_KEY, value)
    navigateUp()
}
```
`previousBackStackEntry`는 NavController의 프로퍼티로 현재 화면 바로 이전 화면의 상태를 저장한다. 
`NavBackStackEntry`에 저장된 각각의 화면들은 자신들의 `savedStateHandle`를 가지는데, 쉽게 말하자면 각 화면마다 작게 자신들만의 데이터 저장소를 가지는 것이다.  

이 함수는 `previousBackStackEntry`를 사용해 이전 화면 `BackStackEntry` 객체의 savedStateHandle 에 접근하여 데이터를 저장하고, `navigateUp()`하여 상위 화면으로 이동, 현재 화면은 Pop한다. 
(여기서 왜 바로 이전 화면으로의 이동을 보장하는 `popBackStack()`이 아닌 `navigateUp()`을 선택했는지 약간 의문이다.)  

**2. `navigateBackWithSerializableResult`: Json 문자열로 인코딩**
```
inline fun <reified T> NavController.navigateBackWithSerializableResult(value: T) {
    navigateBackWithResult(Json.encodeToString(value))
}
```
1번의 확장함수에서 saveStateHandle에서 데이터를 저장할 때, 타입이 원시 타입, 문자열, 배열 등으로 어느정도 제한되어 있다.  
하지만 만약 객체를 넘기고 싶다면? 이 확장 함수를 사용해서 Json으로 직렬화한 뒤에 전달할 수 있다.

이 함수는 inline을 사용해 호출부에 본문을 바로 삽입해 실행된다. 이런 inline의 장점은 함수 호출로 생기는 스택 오버헤드를 줄일 수 있다는 데에 있다.  

`reified` 옵션은 뭘까? 바로 T의 타입을 런타임에도 기억하는 것이다. Json의 `encodeToString` 함수는 다음과 같이 생겼는데,
```
@OptIn(ExperimentalSerializationApi::class)
public inline fun <reified T> Json.encodeToString(value: T): String {
    // 1. reified T 타입을 이용해 해당 타입의 Serializer 탐색
    val serializer = serializersModule.serializer<T>()
    // 2. 찾아낸 serializer와 값을 실제 직렬화 로직 수행
    return encodeToString(serializer, value)
}
```
여기서 집중할 부분은, encodeToString은 value의 타입 T를 가지고 해당 타입의 Serializer를 찾아 직렬화를 수행한다는 것이다.  
그렇기 때문에, `navigateBackWithResult(Json.encodeToString(value))` 에서도 value의 타입이 잘 전달되는게 중요하다.  

하지만, 그냥 <T> 제네릭 타입은 런타임에 타입을 기억하지 못한다. 즉 `Json.encodeToString(value)`로 value를 전달할 때는 value의 타입을 알지 못한다는 것이다. 
이 때 필요한게 `reified` 다. 이 옵션은 런타임에도 T를 기억한다.  

결국, `navigateBackWithSerializableResult` 함수는 `saveStateHandle`에 직접 저장할 수 없는 객체 타입을 Json으로 인코딩해 `navigateBackWithResult`함수에 String으로 전달하는 역할이다.  

**3. `navigateForSerializableResult`: 결과 디코딩**
```
suspend inline fun <reified T> NavController.navigateForSerializableResult(route: Any): T? {
    val result: String = navigateForResult(route) ?: return null
    return Json.decodeFromString(result)
}
```
아래서 설명할 `navigateForResult` 함수를 통해 받은 결과를 디코딩하는 함수다. 여기도 역시 `reified`를 사용해 
Json 문자열을 사용자가 인코딩할 때 사용한 타입으로 변환할 수 있다.

**4. `navigateForResult`: 화면 이동 후 다시 돌아온 경우 결과 전달**
```
suspend fun <T> NavController.navigateForResult(route: Any): T? =
    suspendCancellableCoroutine { continuation ->
        val currentNavEntry = currentBackStackEntry
            ?: throw IllegalStateException("No current back stack entry found")

        navigate(route)

        val lifecycleObserver = object : LifecycleEventObserver {
            override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
                if (event == Lifecycle.Event.ON_START) {
                    continuation.resume(currentNavEntry.savedStateHandle[RESULT_KEY])
                    currentNavEntry.savedStateHandle.remove<T>(RESULT_KEY)
                    currentNavEntry.lifecycle.removeObserver(this)
                }
            }
        }

        currentNavEntry.lifecycle.addObserver(lifecycleObserver)

        continuation.invokeOnCancellation {
            currentNavEntry.savedStateHandle.remove<T>(RESULT_KEY)
            currentNavEntry.lifecycle.removeObserver(lifecycleObserver)
        }
    }
```
`suspendCancellableCoroutine`은 취소 가능한 suspend 함수다. 일반 suspend 함수와의 차이점은, `invokeOnCancellation`로 취소 시 후처리가 가능하다는 점이다. 
이 suspend가 취소되는 경우는, 외부에서 `navigateForResult`를 호출한 코루틴이 `.cancel()`로 취소될 때, compose 환경에서 LaunchedEffect 같은 lifecycle에 종속된 코루틴에서 호출되어 해당 composable이 제거되는 경우(뒤로가기 등) 등이 있다.  

이 함수에서는 취소 시 후처리를 다음과 같이 수행하고 있다.  
1. 화면 `savedStateHandle`의 결과 데이터 제거
2. 화면의 (아래에서 설명할) lifecycleObserver 제거

```
continuation.invokeOnCancellation {
    currentNavEntry.savedStateHandle.remove<T>(RESULT_KEY)
    currentNavEntry.lifecycle.removeObserver(lifecycleObserver)
}
```

다음의 코드는 `currentNavEntry에 현재 화면을 저장하고(BackStack에 누락된 경우 에러 Throw) 새 화면으로 이동(`navigate(route)`)한 뒤 다시 뒤로 돌아올 떄, 현재 화면의 재시작을 감지하고 화면에 `savedStateHandle`에 저장된 데이터를 가져오는 함수다.  
```
val currentNavEntry = currentBackStackEntry
    ?: throw IllegalStateException("No current back stack entry found")

navigate(route)

val lifecycleObserver = object : LifecycleEventObserver {
    override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
        if (event == Lifecycle.Event.ON_START) {
            continuation.resume(currentNavEntry.savedStateHandle[RESULT_KEY])
            currentNavEntry.savedStateHandle.remove<T>(RESULT_KEY)
            currentNavEntry.lifecycle.removeObserver(this)
        }
    }
}

currentNavEntry.lifecycle.addObserver(lifecycleObserver)
```

navigate 후 `suspendCancellableCoroutine`은 반환값이 없으므로 일시 중단(suspended) 상태다. `lifecycleObserver`는 현재 화면의 생명주기를 관찰하고 있고, Event.ON_START(화면이 보이기 시작할때)을 감지한다.
`continuation.resume()`은 현재 suspend 함수를 재개하고 함수의 반환값을 전달한다.  

참고로 `suspendCancellableCoroutine`는 직접 suspend를 구현하는 함수이기 때문에, 블록 안에서 `continuation.resume()`이 호출될 때만 suspend 함수가 결과를 반환한다.

**5. 사용법**
이 Helper 함수의 사용법은 다음과 같다.  
- **목적지** 화면에서는 뒤로가기(혹은 확 시 `navigateBackWithResult(result)`(혹은 `navigateBackWithSerializableResult(result)`) 호출
- **출발** 화면에서는 화면 이동 시 `navigate` 대신 `navigateForResult(route)` (혹은 `navigateForSerializableResult(route)`) 사용

## 출처
- 🧩 [Navigate back with results in Jetpack Compose Navigation](https://www.paleblueapps.com/rockandnull/jetpack-compose-navigation-return-results/)
