## Jetpack Compose로 가장 편리해진 5가지
Android Weekly #739 의 글이다.

### XML과 비교할 때 Jetpack Compose의 장점 5가지
1. 리스트 구현 (여러개의 파일 → 하나의 코드)
   - XML의 RecyclerView의 경우 layout, ViewHolder, Adapter, DiffUtil(리스트 전/후 변경사항 비교) 총 4가지 파일이 필요
   - 보일러플레이트 코드가 많고 특히 DiffUtil의 경우 버그를 유발하기 쉬움 → 항목 비교가 미묘하게 잘못되면 리스트가 깜빡이거나 잘못된 상태로 재사용될 수 있음
   - Compose의 LazyColumn은 간단하고 key 파라미터가 DiffUtil을 완전히 대체함 (직접 비교 X 식별자로 구분하여 더 안전하고 편리함)
  
2. Fade In/Out 애니메이션 (수동 → 자동)
   - XML은 투명도와 애니메이션을 한번에 뒤집을 수 없으므로 알파를 조절한 뒤에 끝에 가시성을 설정해야했음 (Visibility GONE은 애니메이션을 끊고 즉시 수행되므로)
   - Compose의 AnimatedVisibility는 하나의 Boolean으로 설정 가능
   - Compose는 알아서 Content를 트리에 보관해두었다가 종료 애니메이션이 끝나면 제거함

3. 단순해진 레이아웃
   - XML에서 Weight가 있는 중첩 LinearLayout 에서는 자식 뷰의 크기를 반복 측정하는 이중 측정 페널티가 존재 → 레이아웃을 깊게 중첩할수록 화면이 버벅일 수 있으니 ConstraintLayout 사용이 권장됨
   - Compose에서는 설계에 따라 자식 요소를 한 번만 측정하므로 중첩을 구현하기 좋음 → 더 직관적으로 화면 구성 가능  

4. Android로 제한되지 않는 호환성
   - Compose Multiplatform은 같은 컴포저블 함수를 데스크탑, IOS, Web 등 여러 플랫폼에서 공유할 수 있음
   - layout, state handling, design system 등의 코드를 공통화 하여 플랫폼 간 코드 중복을 줄임

5. 뛰어난 이식성
    - 계층 구조를 변경하는 것이 아닌 상태 기반으로 화면을 그리는 것은 SwiftUI, Flutter, React가 동작하는 방식과 동일함
    - 따라서 Compose에 익숙해지면 다른 코드들도 좀 더 쉽게 이해할 수 있음

## 출처
- 🧩 [Five Years of Jetpack Compose: Five Things I Love About It](https://androidadventures.dev/five-years-of-jetpack-compose-five-things-i-love-about-it-684a20c42bac)
