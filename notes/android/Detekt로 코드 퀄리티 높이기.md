## Detekt로 코드 퀄리티 높이기

Android Weekly #683의 **Improve Your Kotlin Code Quality with Detekt in Android**을 읽고 정리한 내용이다.

### Detekt이란?
- **일정한 기준을 유지하며 코딩**할 수 있도록 도와주는 코틀린 코드 분석 툴
- 코드를 실행하지 않고 분석하여 다음과 같은 점들을 찾아낸다.
  - 나쁜 코딩 패턴
  - 코딩 스타일 위반
  - 복잡성이 높은 코드
  - 중복되거나 사용되지 않은 코드
  - 잠재적인 버그
- 결과적으로 프로젝트 전반적으로 **높은 코드 퀄리티와 일관성을 유지**하도록 도와준다.

### 왜 사용하는가?
- 잠재적으로 버그를 일으킬 수 있는 문제점들을 일찍 감지한다.
- 가독성을 향상시킨다.
- 일관성있는 코드 스타일을 강화한다.
- 시간이 줄어든다. 자동으로 코드를 검사하기 때문에 로직이나 구조에만 집중할 수 있다.
- CI/CD 파이프라인에 통합될 수 있다. main 브랜치에 머지되기 전 코드 퀄리티를 보장한다.

### Detekt의 `detekt.yml` 파일
*.yml(.yaml)은 보통 설정(Configuration)을 저장하기 위한 포맷으로 사용된다.  

여기서 설정할 수 있는 규칙의 주요 범주는 다음과 같다.
- **Style**  
  이름 규칙, 코드 포맷팅 (일관된 들여쓰기, 줄바꿈 등), 와일드카드 import (`import com.example.*` 형식을 와일드카드 import라 한다) 등
- **Complexity**  
  긴 함수, 너무 많은 조건 분기, 깊게 중첩된 코드 등
- **Performance**  
  불필요한 오브젝트 생성, 비효율적인 동작 등
- **Potential Bugs**    
  빈 Catch 블록, 절대로 실행되지 않을 코드 (`return`, `break` 뒤에 쓴 코드 등), 중복되거나 불필요한 조건
- **Best Practices**  
  하드코딩된 값(상수를 코드에 직접 넣는 것)을 피하기, `?.`과 같은 safe calls 사용하기, 예외 처리하기

(_정리하면서 좋은 코드에 대해서 다시금 생각해볼 수 있었다._)  

만약 Detekt에 이미 설정된 룰을 조정하고 싶으면 이 파일을 수정하면 된다.  

### 경고를 무시하고 싶을 때는?
`@Suppress` annotation을 사용하자. `@Suppress("MagicNumber")` 이런 식으로 특정 경고를 무시할 수 있고, 
원한다면 `@Suppress("MagicNumber", "FunctionNaming")` 처럼 두 가지 이상의 룰도 무시 가능하다.  

<br>

## 출처
- 🧩 [Improve Your Kotlin Code Quality with Detekt in Android
](https://medium.com/codetodeploy/improve-your-kotlin-code-quality-with-detekt-in-android-135615ab8caf)
