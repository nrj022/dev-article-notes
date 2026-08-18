## SSM을 믿고 Compose의 안정성을 무시하지 말자
Android Weekly #739의 글이다.

### 개요
Strong Skipping Mode(SSM) 모드가 도입되면서 개발자들 사이에 오류가 퍼졌다. "SSM이 있으니 파라미터의 안정성과 상관없이 컴포저블은 알아서 스킵될꺼야! 우린 이제 `ImmutableList`, `@Stable`, `@Immutable`이 더이상 필요없어!"  
하지만 안정성을 무시하고 SSM에만 의존하다가는 최적화 기회를 놓칠 수 있다. SSM을 사용하더라도 특정 상황에서는 Skip이 되어야할 부분이 실제로 Skip되지 않을 수 있다.  

### 먼저, SSM 도입으로 무엇이 달라졌는가?
- 불안정 파라미터를 가진 컴포저블이라도 Recomposition에서 스킵이 가능해졌다:
  - 이전에는 파라미터가 모두 Stable한 요소일때만 재구성을 스킵했다.
  - List<T> 같은 불안정 요소가 포함되면 해당 컴포저블 함수는 내용 변경과 관계없이 무조건 부모 요소가 재구성되면 함께 재구성되었다.
  - SSM은 컴포저블 함수에 Unstable 파라미터가 있어도 스킵할 자격을 준다. (불안정 요소의 메모리 주소가 이전과 같으면 스킵을 가능하게 한다.)
- 자동 람다 메모이제이션:
  - 불안정 요소를 캡쳐하는 람다는 모든 재구성에서 재생성됐었다. 즉, 부모 화면에서 리컴포지션이 발생할 때 무조건 람다가 재생성되고, 자식 컴포저블은 새로운 람다를 인지하고 무조건 재구성이 수행됐다.
  - SSM은 자동으로 모든 람다를 `remember` 블록으로 감싼다. 따라서 불안정 요소가 있는 람다를 가지는 자식 컴포저블이라도, 별다른 변화가 없다면 재구성은 스킵된다.

### SSM이 스킵을 판별하는 방법
- Compose는 각 파라미터의 값을 이전 값과 비교한다 → 불안정한 파라미터는 ===(인스턴스 비교)로, 안정적인 파라미터는 .equals()(값 비교)로
- 만약 인스턴스 비교가 아닌 값를 비교하고 싶다면 @Stable 어노테이션을 사용하면 된다.
- 불안정 요소는 인스턴스가 같아도 값이 달라질 수 있는 위험이 있는데 왜 값 비교가 아닌 인스턴스 비교를 수행할까?
  - 일단 값 비교는 비용이 비쌈
  - 보통 안정 요소는 원시 타입이기 때문에 값 비교가 가볍고, ImmutableList를 쓴다고 해도 해당 타입은 Kotlinx가 내부 구조를 비교에 용이하도록 구성했기 때문에 비교 비용이 굉장히 저렴함
  - 따라서 불안정 요소를 값 비교로 수행하면 값 비교 비용과 재구성 비용이 과도하게 발생함
  - 하지만 아무 비교도 하지 않으면 매번 불필요한 재구성이 발생하는 경우가 있기 때문에 SSM은 인스턴스 비교를 수행하도록 함
  - 현대 개발은 보통 상태 값이 바뀔 때 보통 객체 내부를 수정하기보다 .copy()로 새로운 인스턴스로 교체하는 방식을 많이 사용하기 때문에 이를 가정하고 설계된 방식임
 
### Data Class의 함정
- data class 인스턴스는 본질적으로 stable하다고 오해하곤 한다.
- 하지만 실제로는 모든 프로퍼티가 원시타입, String, @Immutable, @Stable 클래스 등과 같은 변경불가능 or 안정 타입이어야 해당 데이터 클래스를 안정적이라고 간주한다.
```
// ❌ Unstable! Because `tags` is a standard List<String>
data class UserState(
    val id: String,         // Stable
    val name: String,       // Stable
    val tags: List<String>  // Unstable! -> Infects entire UserState!
)
```

- 이 데이터 클래스는 List 때문에 불안정하다. 안정적인 데이터 클래스와 비교하면 다음과 같은 차이가 있다.
  - 안정적인 데이터 클래스 A의 경우 `state.copy(name = "New Name")`가 발생하면 .equal()을 수행한다. name만 변경되었음을 확인하고 **해당 부분만 재구성**에 들어간다.
  - 불안정 데이터 클래스 B의 경우 `state.copy(name = "New Name")`가 발생하면 ===를 수행한다. `old !== new`이므로 일단 재구성에 들어가고, **부모 함수를 하나하나 확인하며 값이 달라졌는지 비교한 뒤 변경된 부분을 재구성**한다.
- 따라서 불안정 데이터 클래스의 경우 불필요하게 내부 코드가 실행되는 비용이 발생한다는 것이다.
- 또한 리스트가 달라지지 않아도 주소 값만 바뀌면 무조건 재구성이 수행되니, 이러한 부분도 오버헤드가 될 수 있다.

### 어떤 문제가 발생할 수 있나?
- 복잡한 UI 계층을 가지는 안드로이드 앱의 경우에는 이런 함정들이 프레임 드랍이나 배터리 소모로 이어질 수 있다.
- 예시로는 복잡한 피드를 가지거나 무한한 리스트를 가진 앱, 오디오나 비디오 플레이어처럼 정말 자주 상태 업데이트가 이루어지는 경우, 깊게 중첩된 컴포저블 트리 등이 있다.

### 재구성 디버깅, 안정성 툴들
- Compose Compiler Reports: [Compose Compiler Metrics & Reports](https://developer.android.com/develop/ui/compose/performance/stability/diagnose) 안정성 상태를 검사할 수 있다.
- Compose Stability Analyzer: [Compose Stability Analyzer](https://github.com/skydoves/compose-stability-analyzer) 코딩하면서 실시간으로 피드백을 받을 수 있다.

### 참고: Skipping은 무료가 아니다
- Skip을 수행하기 위해서는 Compose 컴파일러는 컴포저블 함수 내에 스킵용 검사 코드를 많이 집어 넣는다 (스킵을 위해서 이전 값과 비교가 필요하기 때문에)
- 따라서 정말 드물지만 정말 초 단순 layout + massive state data class 조합은 오히려 @NonSkippableComposable로 스킵을 수행하지 않는게 더 저렴할 수 있다

### 결론
- Strong Skipping은 안정망이지만 완전한 해결책은 아니다: 컴포저블을 건너뛸 수 있게 만들지만 불안정 파라미터는 참조 동일성(===) 검사로 대체된다.
- List<T>는 여전히 불안정하다: `copy(), `map {}`, `filter {}`, or `toList()`을 통한 변화는 주소 참조를 변경시키고 state emission마다 재구성을 강제한다.
- 안정성 주석과 ImmutableList는 여전히 중요하다: kotlinx-collections-immutable(ImmutableList<T>)을 사용하거나 @Immutable / @Stable (리스트의 크기가 작을 경우)로 사용자 정의 상태 클래스에 주석을 달아 진정한 구조적(.equals()) Skipping을 활성화하자.

## 출처
- 🧩 [Stop Ignoring Compose Stability (Yes, Even with Strong Skipping Mode)](https://blog.shreyaspatil.dev/stop-ignoring-compose-stability-yes-even-with-strong-skipping-mode/)
