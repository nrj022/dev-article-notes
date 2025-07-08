## Jetpack Navigation 3 출시

### 새로운 Navigation이 필요했던 이유?
가장 큰 한계 중 하나는, 백스택(히스토리)를 간접적으로만 접근할 수 있다는 점이다. <br>

- 한 가지 예로, 개발자는 백스택에 직접 접근할 수 없으므로 현재 화면을 추적해야한다. 
- 여기서 문제점은 추적한 화면과 실제 시스템의 백스택의 상태 불일치가 발생할 수 있다는 점이다.

또한, Navigation 2의 `NavHost`는 백스택 최상단에 위치한 단일의 화면만을 Display 하도록 설계되었다. <br>

- 이로 인해 다양한 화면 크기를 고려한 적응형 레이아웃 구현이 어려워진다. 
- 예를 들어, 갤럭시 탭에서 큰 화면을 활용하기 위해 단일 리스트 패널 대신
  화면을 나누어 리스트와 컨텐츠를 동시에 Display 하는 것은 Navigation 2에서는 복잡한 과정을 거쳐야 한다. (2개의 `NavHost`를 사용하는 등)

<br>

### Navigation 3 특징
- 개발자에 의해 Control 되는 Backstack
- Compose 같은 State-based 모델

<br>

### Event-based인 Old Navigation Compose library의 비동기적 불일치 현상

1. 사용자가 화면 전환 버튼 클릭
2. Split-Second Confusion(찰나의 혼란):<br/>
   ViewModel과 같은 상태 관리 컴포넌트는 즉시 상태 업데이트
   하지만, 실제 화면을 바꾸는 Navigation 라이브러리는 아직 이전 화면 상태에 머뭄
3. 잠깐 동안 앱의 상태와 Navigation 라이브러리의 상태가 불일치
4. Late Catch-Up:<br/>
   Navigation 라이브러리가 이동 이벤트 처리, 실제 화면 전환,
   이 시점이 되어야 Navigation 상태와 앱 상태 일치

이 작은 싱크 불일치가 예측하지 못한 Behaviour와 디버깅의 스트레스를 유발

<br>

### Navgation 3의 화면 전환

1. 사용자가 화면 전환 버튼 클릭
2. App의 State가 변경됨
3. Navigation 3는 지속적으로 App State 확인, State 변경 시 자동으로 변경된 화면을 보여줌

<br>

## 참고 자료
- 🧩 [Announcing Jetpack Navigation 3](https://android-developers.googleblog.com/2025/05/announcing-jetpack-navigation-3-for-compose.html)
- 🧩 [Future Of Android: Why Navigation 3 is a Game-Changer!](https://proandroiddev.com/future-of-android-why-navigation-3-is-a-game-changer-f835f841c17f)
