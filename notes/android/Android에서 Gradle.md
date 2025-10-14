## 안드로이드 스튜디오와 항상 함께하는 Gradle, 역할은?

### Android Studio의 필수 동작 요소

| 구성 요소 | 역할 |
|------------|------|
| **Gradle + Android Gradle Plugin(AGP)** | 프로젝트를 빌드, 의존성 관리, APK 생성 등을 담당 |
| **Android SDK** | 앱 개발용 라이브러리(Activity, View 등), 도구(Emulator 등) 제공, 개발 도구 모음 |
| **Kotlin/Java 컴파일러** | Kotlin/Java 코드 → JVM 바이트코드(.class) → *DEX 바이트코드 변환 |
| **IDE(안드로이드 스튜디오)** | 코드 편집, 자동완성, UI 디자인 미리보기 등 |

> 💡 *DEX 바이트코드*: Android 앱은 보통 JVM이 아닌 ART(Android Runtime) 가상 머신 위에서 동작하기 때문에 DEX 바이트코드로의 변환 과정이 필요하다.

### Android Studio 동작 과정 요약

1. Kotlin/Java 코드 작성 (IDE, Android SDK 활용)  
2. Gradle + AGP가 의존성을 관리하고 빌드(APK 생성) 계획을 생성  
3. APK 빌드 시 필요한 작업(의존성 확인, 컴파일 등)을 올바른 순서로 배치  
4. 프로젝트에 필요한 라이브러리, SDK 등을 확인하고 로컬에 없는 경우 다운로드  
   - AGP는 빌드 계획을 **짜고**, Gradle은 그 계획을 **실행 및 의존성 관리**  
5. 컴파일 및 DEX 변환  
6. 리소스(XML, 이미지 등) 처리 + 패키징 → APK 생성  

> Gradle은 안드로이드 프로젝트 빌드에 없어서는 안 될 **매니저**로,  
> 주요 역할은 프로젝트 의존성을 관리(확인 후 다운로드)하고  
> AGP가 정의한 빌드 계획을 순서대로 실행하는 것이다.  
> 직접 코드를 컴파일하거나 APK를 만드는 것은 아니며, 대신 관련 도구에 **작업을 지시하는 역할**을 한다.

## Android 프로젝트 생성 시 함께 생성되는 Gradle 관련 파일

### 1. `libs.versions.toml`

프로젝트에서 사용되는 **의존성들의 버전 관리**를 한 번에 정리할 수 있는 파일이다.

#### 구성 블록
- `[versions]`: 버전 번호만 정리  
- `[libraries]`: 의존성(라이브러리) 정의  
- `[plugins]`: Gradle 플러그인 버전 정의  

#### 문법 요약
- `version.ref` → `[versions]` 블록에서 버전 참조  
- `group`, `name`, `version` → 라이브러리 식별자  
- `id` → Gradle 플러그인용  
- `alias` → TOML의 이름 자동 매핑  

#### 예시
`coil-compose = { group = "io.coil-kt", name = "coil-compose", version = "2.6.0" }`  
→ 각 모듈 `build.gradle.kts`에서  
`implementation(libs.coil.compose)` 로 사용 가능하며,  
이는 `implementation("io.coil-kt:coil-compose:2.6.0")` 와 동일하다.

### 2. `build.gradle.kts (Project)`

루트 수준의 `build.gradle.kts`는 **프로젝트 전반에 공통으로 사용하는 플러그인 버전**을 등록한다.  
루트 빌드는 **안드로이드 앱이 아니므로** 실제 플러그인을 적용하지 않고 `apply false` 로 선언만 한다.

#### 주요 플러그인
| 플러그인 | 역할 |
|-----------|------|
| `alias(libs.plugins.android.application)` | Android Gradle Plugin (AGP). 앱 모듈이 실행 가능한 Android 앱임을 인식. `android { ... }` DSL 제공 |
| `alias(libs.plugins.kotlin.android)` | Kotlin 코드가 Android 프로젝트 내에서 동작하도록 함. Kotlin 컴파일러 옵션 지원 |
| `alias(libs.plugins.kotlin.compose)` | Compose 기능용 Kotlin 컴파일러 확장 플러그인. `@Composable`, Preview, recomposition 등 지원 |

### 3. `build.gradle.kts (Module)`

#### [plugins]
이 모듈이 어떤 빌드 플러그인을 사용하는지 선언하는 부분.

#### [android]
안드로이드 빌드 관련 설정 담당

- **namespace**: 패키지 고유 이름 설정  
- **compileSdk**: 어떤 Android SDK 버전으로 컴파일할지 지정  
  > 최신 버전 유지 권장. 낮으면 최신 API 사용 시 컴파일 에러 발생 가능.

#### defaultConfig: 앱의 기본 빌드 설정

| 항목 | 설명 |
|------|------|
| `applicationId` | 앱 고유 식별자 (배포 시 패키지 이름) |
| `minSdk` | 설치 가능한 최소 Android 버전 |
| `targetSdk` | 정상 동작을 보장하는 Android 버전 |
| `versionCode` | 내부 빌드 버전 (숫자) |
| `versionName` | 사용자에게 보이는 버전 (예: `"1.0"`) |
| `testInstrumentationRunner` | Android 테스트 실행기 지정 |

#### Android SDK (Software Development Kit) ? 
- 안드로이드 개발에 필요한 도구와 라이브러리 모음

> 중요한 점:  
> SDK에 포함된 API는 **앱을 컴파일할 때 참조용 정의**일 뿐,  
> 실제 동작은 **Android OS에 구현된 API**를 통해 수행된다.  
> 즉, 앱이 실행될 때는 OS가 제공하는 실제 구현체를 사용한다.

- 참고
  - Android OS의 각 버전은 제공하는 **API 레벨(SDK 버전)** 이 다르다.  
  - 앱 설치 시, 기기의 OS가 앱의 **minSdk 이상**을 지원해야 한다.  
  - 오래된 기기에서도 모든 앱을 실행할 수 있는 것은 아니다.  
    제조사가 **OS 업데이트 지원을 제한**하기 때문.  
    - 예: 삼성은 OS 지원을 **4년 → 7년**으로 연장함.

#### buildTypes: 빌드 환경별 설정

| 항목 | 설명 |
|------|------|
| `isMinifyEnabled` | 코드 난독화 여부 |
| `proguardFiles` | ProGuard/R8 규칙 파일 지정 |
| `getDefaultProguardFile(...)` | 안드로이드 기본 최적화 규칙 포함 |

#### compileOptions, kotlinOptions
- Kotlin과 Java의 컴파일 옵션 지정  
- 바이트코드 타깃 JVM 버전 설정 (어떤 JVM 버전을 대상으로 컴파일할지 지정)

#### buildFeatures
빌드 시 활성화할 기능 지정 (예: Compose, ViewBinding 등)

#### [dependencies]
- 모듈에서 필요한 라이브러리 선언  
- Gradle이 자동으로 다운로드하고 빌드에 포함시킴

**+ 내용 추가 예정**
