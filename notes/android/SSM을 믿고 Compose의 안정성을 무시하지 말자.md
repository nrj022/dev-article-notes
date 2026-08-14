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

## 출처
- 🧩 [Stop Ignoring Compose Stability (Yes, Even with Strong Skipping Mode)](https://blog.shreyaspatil.dev/stop-ignoring-compose-stability-yes-even-with-strong-skipping-mode/)
