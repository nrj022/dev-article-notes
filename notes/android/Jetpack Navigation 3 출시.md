## Navigation 3 특징

- 개발자에 의해 Control 되는 Backstack
- Compose 같은 State-based 모델

## Event-based인 Old Navigation Compose library의 비동기적 불일치 현상    

1. 사용자가 화면 전환 버튼 클릭
2. Split-Second Confustion(찰나의 혼란):<br/>
   ViewModel과 같은 상태 관리 컴포넌트는 즉시 상태 업데이트
   하지만, 실제 화면을 바꾸는 Navigation 라이브러리는 아직 이전 화면 상태에 머뭄
3. 잠깐 동안 앱의 상태와 Navigation 라이브러리의 상태가 불일치
4. Late Catch-Up:<br/>
   Navigation 라이브러리가 이동 이벤트 처리, 실제 화면 전환,
   이 시점이 되어야 Navigation 상태와 앱 상태 일치
<br/>

- 이 작은 싱크 불일치가 예측하지 못한 Behaviour와 디버깅의 스트레스를 유발

## Navgation 3의 화면 전환

1. 사용자가 화면 전환 버튼 클릭
2. App의 State가 변경됨
3. Navigation 3는 지속적으로 App State 확인, State 변경 시 자동으로 변경된 화면을 보여줌

## 참고 자료
- 🧩 [Announcing Jetpack Navigation 3](https://android-developers.googleblog.com/2025/05/announcing-jetpack-navigation-3-for-compose.html)
- 🧩 [Future Of Android: Why Navigation 3 is a Game-Changer!](https://proandroiddev.com/future-of-android-why-navigation-3-is-a-game-changer-f835f841c17f)
