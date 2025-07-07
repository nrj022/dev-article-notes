## KMP에서의 알림 구현을 위한 새로운 라이브러리 Alarmee

### Kotlin Mutliplaform(KMP)가 뭘까?
- 안드로이드와 iOS 앱을 하나의 공통 코드베이스로 개발할 수 있도록 해주는 JetBrains에서 제작한 기술
- KMP는 각 플랫폼에 최적화된 네이티브 성능을 유지하면서, 공통 코드를 통해 크로스 플랫폼의 생산성도 제공하는 하이브리드 접근 방식이다.
- UI 등 플랫폼 고유 기능은 각각 플랫폼별로 구현하고, 나머지 공통 로직은 공통 모튤에서 작성해 공유한다.
- ❗UI까지 완전히 공통화하고 싶다면 Compose Multiplatform (CMP)를 함께 사용한다. _이는 추후에 정리해볼 예정이다._

### KMP에서의 알림 기능 구현
KMP를 사용할 때 가장 큰 과제 중 하나는 **알림** 같은 플랫폼 종속적인 기능이다.
- Android와 iOS의 구현 방식 및 처리 방식이 달라
- 공통 코드에서 이를 직접 다루기 어렵기 때문

결과적으로 복잡한 플랫폼 분기 처리와 중복 코드가 생기기 쉬운 구조다.

### 새로운 라이브러리 Alarmee
- Alarmee는 KMP, CMP에서 로컬 및 푸시 알림을 쉽게 다룰 수 있도록 설계된 라이브러리
- 1회성 알람, 반복 알람 및 푸시 알림 등을 공통 코드에서 제어할 수 있도록 지원한다.
- 기존 KMP 알림 구현 시 발생하는 복잡함을 줄이고, 플랫폼별 맞춤 설정도 쉽게 처리할 수 있다.

## 참고 자료
- 🧩 [Alarmee: Schedule Local and Push Notifications in KMP](https://vivienmahe.medium.com/alarmee-schedule-local-and-push-notifications-in-kmp-44ea47972ae7)
- 🤖 Chat GPT
