## Compose 안정성을 위한 팁

Android Weekly #684의 'Compose stability tips and tricks'를 읽고 정리한 글이다.

### 공식 문서가 정의한 Stable 타입은 ?
**1. 변하지 않는 불변 타입 (Immutable)  
2. 또는, 값의 바뀔 때 알 수 있는 타입 (ex. State)**  
  
만약 값이 바뀌는 것을 알 수 없다면 어떤 일이 일어날까?
다음 예를 보며 생각해볼 수 있다.
```
@Composable
fun Parent(hero: Hero) {
  Child(hero)
}

@Composable
fun Child(hero: Hero) {
  Text(hero.name)
}
```
만약 **`Hero`가 불안정 타입**이라면, Compose는 `hero`가 언제 바뀔지 예측할 수 없다.
따라서, `Parent`가 재구성될 때 Compose는 `hero`의 불안정성 때문에 **`Child`도 무조건 재구성**한다.  
하지만 만약 **`Hero`가 안정적 타입**이었다면, Compose는 `hero`를 이전 값과 비교하고, 
변경이 없는 경우 **`Child` 재구성을 스킵**한다.  

결국 안정적인 타입을 사용하는 것은 Compose의 불필요한 Recomposition을 줄일 수 있다.

### class의 안정성 ?
1. 이 클래스는 안정할까?  
```
data class Hero(val name: String, val power: String)
```
변경 불가능한 참조 타입인 String을 사용하고, val 프로퍼티 사용으로 이 값은 변경될 수 없다.  
따라서 이 데이터 클래스는 immutable 하므로 **stable**하다고 볼 수 있다.  

2. 하지만 만약, `var` 프로퍼티를 사용한다면,
```
data class Hero(var name: String, var power: String)
```
이 데이터 클래스의 필드는 객체가 생성된 이후에도 변할 수 있다. 이 때, 필드 타입을 `State`로 감싸지 않으면 Compose는 값이 바뀌어도 알지 못한다.  
따라서 이 데이터 클래스는 **unstable**하다.  

3. 변경 가능하지만 **stable**한 클래스는?
```
class Hero(
    name: String,
    power: String
) {
    var name by mutableStateOf(name)
    var power by mutableStateOf(power)
}
```
이 `Hero` 클래스의 프로퍼티는 변경 가능하지만, `MutableState<T>` 타입은 변경 시 Recomposition을 유도하므로 이 클래스는 **stable**하다.

### 타입의 안정성을 확인하는 방법
- Compose의 **Compiler Reports**를 확인하자.
    
  `build.gradle`에서 3가지 report를 생성하도록 설정할 수 있다. (아래 출처의 공식 문서에서 설정 방법을 참고)  
  3가지 report 중 우리는 **클래스의 안정성을 확인할 수 있는 `modulename-classes.txt`를 사용**한다. (_이래서 프로젝트를 모듈화 하는 걸지도 모르겠다. 코드를 여러 모듈로 나누는게 확실히 검증하는데 용이할 것 같다._)  

  이 파일을 통해 우리는 다음을 확인할 수 있다:
  1. class가 stable 한지
  2. 어떤 프로터피가 stable/unstable한지
  3. 무엇이 안정성의 원인이 되는지

  Compose Compiler를 사용해 **현재 진행 중인 프로젝트 모듈의 리포트를 확인**해보았다. 
   ```
  stable class DatePicker {
  <runtime stability> = Stable
  }
  stable class DateRangePicker {
    <runtime stability> = Stable
  }
  unstable class PeriodDataModel {
    unstable val startDate: LocalDate
    unstable val endDate: LocalDate
    <runtime stability> = Unstable
  }
  ```
  여기서 왜 다음의 `PeriodDataModel`은 불안정할까?
  ```
  data class PeriodDataModel (
    val startDate: LocalDate,
    val endDate: LocalDate
  )
  ```
  
  필드들이 모두 `val`프로퍼티를 사용함에도 Compose가 이 데이터클래스가 불안정하다고 판단했다.
  
  이건 ChatGPT에게 물어본 결과, **Compose가 외부(Java)의 타입에 대해 보수적으로 보기 때문**이라고 한다. `LocalDate` 자체가 mutable이 아님에도 Compose 입장에서는 **안정성 확인이 불가능하여 unstable하게 간주**한다고 한다. 이렇게 되면 Compose가 불필요한 Recomposition 발생 가능성이 있다.

  이런 경우, @Immutable 을 사용해 컴파일러에게 변경되지 않는 클래스임을 알려줄 수 있다.
  ```
  @Immutable
  data class PeriodDataModel(
      val startDate: LocalDate,
      val endDate: LocalDate
  )
  ```
  하지만 이번엔 클래스는 stable 하지만 그 안의 필드가 unstable이라고 판단했다. 여기도 여전히 불필요한 Recomposition이 발생 가능하다.
  ```
  stable class PeriodDataModel {
    unstable val startDate: LocalDate
    unstable val endDate: LocalDate
  }
  ```

  해결책은, 아래와 같이 LocalDate를 모두 감싸거나,
  ```
  @Immutable
  data class ImmutableLocalDate(val value: LocalDate)
  
  @Immutable
  data class PeriodDataModel(
      val startDate: ImmutableLocalDate,
      val endDate: ImmutableLocalDate
  )
  ```
  타입을 원시(Primitive) 타입으로 변경하는것이다.
  ```
  data class PeriodDataModel (
    val startDate: String,
    val endDate: String
  )
  ```
  두 번째 방법을 사용해 코드를 수정해보니, stable한 클래스를 얻었다.
```
  stable class PeriodDataModel {
    stable val startDate: String
    stable val endDate: String
    <runtime stability> = Stable
  }
```
  만약 성능을 예민하게 고려한다면 이런 부분을 신경써서 코드를 작성하면 좋을 것 같다.
  
**+ 내용 추가 예정**

## 출처
- 🧩[Compose Stability tips and tricks](https://leedwon.github.io/posts/Compose-stability-tips-and-tricks/#stability-in-a-nutshell)
- 🧩[Compose Compiler 관련 공식 문서](https://developer.android.com/develop/ui/compose/compiler)
