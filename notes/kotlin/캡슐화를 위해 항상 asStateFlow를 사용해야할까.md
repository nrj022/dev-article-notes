## 캡슐화를 위해 항상 asStateFlow를 사용해야할까?

Android Weekly #687 **"Hot take : When Encapsulation Becomes Ceremony (캡슐화가 형식적인 절차로 전락할 때)"** 를 읽고 작성하는 글이다. 
글의 내용과 함께 캡슐화와 asStateFlow에 대해서도 정리해봤다.

### 캡슐화 (encapsulation)?
객체 지향 프로그래밍에서 객체에서 **외부에 필요한 기능만 공개하고 내부 구현은 숨겨 외부에서 접근하거나 확인할 수 없도록 하는 것**이다. 
캡슐화를 통해서 우리는 외부로부터 내부 상태가 예상치 못하게 변경되는 것을 막고, 내부 구현의 변화에도 외부에 그 영향이 적도록 합니다.

### asStateFlow() vs StateFlow<T> 타입 캐스팅
두 방식 모두 캡슐화에서 할 때 다음과 같이 자주 사용된다.
```
// StateFlow<T> 타입 캐스팅
private val _uiState = MutableStateFlow(UiState())
val uiState: StateFlow<UiState> = _uiState

// asStateFlow()
private val _uiState = MutableStateFlow(UiState())
val uiState = _uiState.asStateFlow()
```
두 방식의 목적은 모두 `_uiState`를 Mutable 하지 않은 read only 타입으로 변경하여 외부 접근을 제한하는 데에 있다.  

하지만 두 방식이 read only 하게 데이터에 접근하는 방식은 서로 다른데,  

1. StateFlow<T> 타입 캐스팅은 **`_uiState`의 참조를 StateFlow 타입으로 선언하는 방식**이다.
   
    이게 가능한 이유는 MutableStateFlow가 StateFlow의 하위 타입이기 때문이다.  
  
    _* 타입 캐스팅은, 간단하게 같은 객체의 메모리 주소를 가르키는 변수들이 서로 다른 타입으로 객체를 바라보는 것이다._
    
    그렇기 때문에 **표면적으로 `uiState`는 StateFlow 타입으로 read-only이지만 실제 객체는 mutableStateFlow** 인 것이다.  
    이 방식의 취약 포인트는, **실제 객체가 mutableStateFlow 이기 때문에 toMutableStateFlow()로 타입을 강제로 캐스팅하여 값을 변경할 수 있다**는 점이다. 

3. asStateFlow()는 타입 캐스팅과 다르게, **`_uiState`를 StateFlow 타입으로 감싸 새로운 래퍼 객체를 반환**한다.     
  
    `uiState`는 **`_uiState` 주소에서 값을 가져오는 함수만 구현되어있는 read-only한 StateFlow 객체**이기 때문에, 실제 MutableStateFlow 객체의 값을 직접 변경시킬 수 없다.  

따라서, StateFlow<T> 타입 캐스팅 방법은 asStateFlow()에 비해 구현이 간단하고 래퍼 객체를 생성하지 않는다는 장점이 있지만, 그만큼 안정성이 떨어진다.

### 아티클 작성자가 말하는 것: `asStateFlow()`를 사용한 불필요한 캡슐화는 지양하자!
두 방식의 장단점으로만 보면 안정성을 위해서는 asStateFlow()를 사용하는 것이 좋은 방법인 것 같아 보인다.  

하지만 타입 캐스팅으로 변환한 read-only 변수를 toMutableStateFlow()로 강제 캐스팅하여 값을 변경하는 것은 기술적으로 가능하지만 
이러한 **불필요하고 우회하는 방식을 실무자가 선택한다는 것은 상식적으로 발생할 수 없다**는 것이다. 
작성자는 그런 사람이 있다면 그건 안정성을 위해 코드를 수정해야할 문제가 아니라 그 사람을 해고하거나 가르쳐야할 문제라고 말한다.  

결론적으로, 작성자는 이런 '혹시나'하는 마음으로 불필요한 로직(래핑)을 추가하는 것은 **더이상 방어적인 코드를 짜는 것이 아니라 '의례적인' 코드를 짜는 것이라고 강조**하며,
그런 **의례적인 'asStateFlow()' 사용은 지양하자고 주장**한다.

_물론, asStateFlow()가 필요없는 함수라는 것은 아니다. 공개적인 SDK에서는 asStateFlow()가 그 진가를 발휘한다._

## 출처
- 🧩 [Hot take : When Encapsulation Becomes Ceremony](https://yveskalume.dev/hot-take-when-encapsulation-becomes-ceremony)
