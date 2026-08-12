## TestBalloon로 구현하는 스냅샷 테스트 메트릭스

Android Weekly #739 "The Snapshot-Test Matrix, Two Ways: Parameterized JUnit vs TestBalloon"를 읽고 작성하는 글이다.

### 수많은 화면 상태의 테스트를 JUnit으로 구현하지 않는 이유
- 작성자의 앱은 확인해야할 화면 상태가 27가지이고, 각각은 5가지 Config로 렌더링된다 — 기본값, 다크모드, 테블릿, 폰트 크기 등 (다크모드로는 하드코딩된 색상을 잡아내고, 폰트 크기는 레이아웃이 무너지는 것을 잡음)
- 결국 총 135가지의 상태가 있다는 것인데 이걸 하나하나 테스트를 작성해야할까?
- JUnit의 어노테이션은 이걸 표현하지 못한다. Config 어노테이션은 컴파일 시점에 확정된 상수를 사용해야하기 때문에 변수나 함수를 사용해서 동적으로 환경을 설정할 수 없고, Looping도 할 수 없다.

### 더 편리하게 구현할 수 있는 두 가지 방식을 비교해보자
**Robolectric & Roborazzi**
- 참고: [Efficient Testing with Robolectric & Roborazzi Across Many UI States, Devices and Configurations](https://sergiosastre.hashnode.dev/efficient-testing-with-robolectric-roborazzi-across-many-ui-states-devices-and-configurations)
- `ParameterizedRobolectricTestRunner`를 통해 행렬을 공급하고 런타임에 config를 적용할 수 있다.

```
@RunWith(ParameterizedRobolectricTestRunner::class)  // 로보렉트릭의 파라이미터화 전용 엔진으로 테스트 구동
class CoffeeDrinkSnapshotTest(private val testItem: TestItem) {  // TestItem: 화면 상태(27개 중 하나)와 Config(5개 중 하나)가 들어있음

    companion object {
        // 화면 상태, 기기, 설정을 코드로 곱해서 모든 매트릭스 조합을 만들어냄
        @JvmStatic
        @ParameterizedRobolectricTestRunner.Parameters(name = "{0}")  // JUnit 엔진이 각 테스트별로 독립적으로 실행하도록 함. 각 조합의 이름은 첫번재 파라미터 값으로 깔끔하게 표시됨
        fun testData() = /* states × devices × configs as TestItems */
    }

    // 어노테이션을 쓰지 않고 실제 코드를 사용하여 다양한 설정을 동적으로 모두 적용 가능
    @Test
    fun snapshot() {
        RuntimeEnvironment.setQualifiers(testItem.deviceQualifier)
        RuntimeEnvironment.setFontScale(testItem.config.fontScale)
        captureRoboImage(testItem.screenshotId) {
            CoffeeDrinkListItem(drink = testItem.coffeeDrink.uiState)
        }
    }
}
```
- 하지만 이 방식은 프레임워크가 유연하지 않기 때문에 프레임워크에서 정해진 룰에 따라 억지로 만드는 결과물에 가깝다.

**The TestBalloon DSL**
- 코틀린 멀티플랫폼 테스트 프레임워크
- 결과물을 간단하게 표현하는 방식으로 구현 가능
- 따라서 리스트 요소에 대한 테스트는 그냥 루프로 구현할 수 있고 행렬을 DSL 형태로 작게 표현 가능하다.

```
val MainScreenSnapshotTests by testSuite { 
    snapshotSuite<MainScreenSnapshotContent>("MainScreen") 
}

class MainScreenSnapshotContent(variant: String) :
    SnapshotSuiteContent<MainState>(
        goldenPrefix = "main_screen",
        variantName = variant,
        render = { state -> MainScreen(state = state, onAction = {}) },
        scenarios = {
            scenario("locations_list", MainState(locations = listOf(albertHeijn, jumbo)))
            scenario("loading", MainState(isLoading = true))
            scenario("error", MainState(error = MainError.Network))
            // ... seven more states
        },
    )
```

- 5가지 변수(variant)는 앱 전체에서 하나의 enum으로만 살아있고 각각은 기본값으로부터 하나의 축만 변경한다.

```
enum class SnapshotVariant(
    val qualifiers: String, 
    val fontScale: Float, 
    val isDark: Boolean = false
) {
    BASELINE(qualifiers = "+en-w411dp-h891dp", fontScale = 1.0f),
    DARK(qualifiers = "+en-w411dp-h891dp-night", fontScale = 1.0f, isDark = true),
    FONT_SCALE(qualifiers = "+nl-w411dp-h891dp", fontScale = 2.0f),
    // LOCALE, TABLET
}
```
- 10줄로 구성된 함수가 이 조합을 곱한다.
- 각 변경 버전마다 Robolectric 테스트 스위트를 등록하는데, 이 스위트의 가상 환경은 켜지기도 전에 TestBalloon의 `TestConfig.robolectirc`를 통해 SDK, qualifiers, font scale를 세팅을 완료한다.
- 따라서 실행중인 테스트 내부에서 `setQualifiers`를 호출해 상태를 지저분하게 변경할 필요가 없다.
- 테스트 결과 보고서는 일렬로 길게 늘어진 135개의 문자열 이름 대신, MainScreen snapshots (dark) → error와 같은 트리 구조로 출력된다.
- 덕분에 각 노드(node)별로 테스트를 개별적으로 다시 실행하거나 필터링할 수 있다.
  
## 포함되는 개념
**Snapshot Test?**
- 프로그램의 현재 결과물이 사전에 저장해 둔 기준값과 일치하는지 비교하여 의도하지 않은 변경이나 버그를 찾아내는 테스트 기법
- Matrix의 형태를 가짐
  
**Robolectric(로보렉트릭)?**
- 안드로이드 에뮬레이터나 실제 기기 없이, 컴퓨터(JVM) 환경에서 안드로이드 유닛 테스트를 직접 실행할 수 있게 해주는 오픈소스 테스트 프레임워크
- 에뮬, 스마트폰 연결 없이 빠르게 테스트 가능
- 코드를 통해 버튼 존재나 화면 이동 여부 등을 테스트 가능 (텍스트와 데이터 형태의 상태 검증)

**Roborazzi(로보라치)?**
- 로보렉트릭 환경 위에서 동작, 테스트가 실행되는 순간의 화면을 캡쳐(스냅샷)해서 이미지로 만듦

## 출처
- 🧩 [The Snapshot-Test Matrix, Two Ways: Parameterized JUnit vs TestBalloon](https://emartynov.medium.com/the-snapshot-test-matrix-two-ways-parameterized-junit-vs-testballoon-d4015c495efc)
