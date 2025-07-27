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
- Compose의 Compiler Reports를 확인하자.
    
  `build.gradle`에서 3가지 report를 생성하도록 설정할 수 있다. (아래 출처에서 설정 방법을 참고)  
  3가지 report 중 우리는 클래스의 안정성을 확인할 수 있는 `modulename-classes.txt`를 사용한다.  

  이 파일에서 우리는
  1. class가 stable 한지
  2. 어떤 프로터피가 stable/unstable한지
  3. 무엇이 안정성의 원인이 되는지  
  를 확인할 수 있다.



**+ 내용 추가 예정**

## 출처
- 🧩[Compose Stability tips and tricks](https://leedwon.github.io/posts/Compose-stability-tips-and-tricks/#stability-in-a-nutshell)
