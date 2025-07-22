## Jetpack Compose Navigation 라이브러리

공식 문서 **Navigation in Compose**를 읽고 정리한 글이다.  

*아래 **구현 시 고려사항**을 읽고 진행하고 있는 'RentIt'프로젝트에서는 화면별로 뷰모델을 분리해 각 책임을 명확하게 하도록 리팩토링했다.

### 기본적인 Navigation 문제를 해결
- 화면 이동 과정에서 Type-safe로 데이터를 전달한다.
  
  <pre><code>
  navArgument("userId") {
    type = NavType.IntType
  }
  </code></pre>
- Navigation History를 명확하고 접근하기 쉽도록 하여 사용자의 이동 동선을 추적하기 쉽도록 한다.
- Deep Link 기능을 지원하여 사용자는 어플 외부에서 링크를 통해 어플의 특정 화면으로 바로 이동할 수 있다.

### 네비게이션 라이브러리의 주요 클래스
- `NavController`<br>
  핵심 네비게이션 기능 API들을 제공한다. 이는 화면(목적지)간 이동, Deep Link 처리, Back Stack 관리 등을 포함한다.
- `NavHost`<br>
  네비게이션 그래프에 따라 현재 목적지에 해당하는 화면 콘텐츠를 표시하는 Composable(UI를 그릴 수 있는 함수)이다.
  사용자가 처음으로 보게될 화면을 설정하기 위해 `startDestination` 파라미터를 필수로 요구한다.
- `NavGraph`<br>
  앱에 존재하는 모든 화면(목적지)와 화면들 사이를 어떻게 이동할 수 있는지 나타내는 그래프 구조이다.
  Kotlin 코드의 람다 블록 안에서 보통 다음과 같이 정의한다.
  
  <pre><code>
  NavHost(navController, startDestination = "home") {
    composable("home") { HomeScreen() }  // NavGraph를 구성하는 람다
    composable("detail") { DetailScreen() }
  }
  </code></pre> 

### 구현 시 고려사항
화면간 최소한의 필요한 정보만 전달해야 한다. 상태를 반영하는 복잡한 오브젝트나 파일 등은 데이터 계층(ViewModel)에서 보관되어야 한다.<br><br>
예를 들어,
- 전체 사용자 프로필은 전달하지 말 것. User ID를 전달해 목적지에서 프로필을 로드할 것.
- 이미지 오브젝트를 전달하지 말 것. URI나 파일 이름을 전달해 목적지에서 이미지를 로드할 것.
- 뷰모델이나 상태를 전달하지 말 것. 화면이 작동하는 데 필요한 정보만 전달할 것.

## 출처
- 🧩 [Navigation in Compose](https://www.jetbrains.com/help/kotlin-multiplatform-dev/compose-navigation.html)
