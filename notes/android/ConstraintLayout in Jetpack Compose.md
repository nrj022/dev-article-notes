## ConstraintLayout 란?
Android에서 UI를 구성할 수 있는 Layout 중 하나로, 뷰 간의 상대적인 제약 조건을 사용한다.
생성된 UI는 중첩이 적고 Flat한 뷰 계층 구조로 이루어지며, 이는 복잡한 UI에서 더 나은 성능과 깔끔한 구조를 가지도록 한다.

## Jetpack Compose의 ConstraintLayout
기본값으로 `ConstraintLayout`을 가지지는 않으니, 의존성 추가가 필요하다.
<pre><code>
implementation("androidx.constraintlayout:constraintlayout-compose:1.0.1")
</code></pre> 

### 사용법
1. 각 컴포저블의 Reference 생성 (`createRefs()` 또는 `createRefFor()`)
   <pre><code>
     val (profileImage, name, tagline, followBtn) = createRefs()
   </code></pre>
3. Modifier의 `constrainAs()` 옵션으로 제약 조건 추가 가능
   - 파라미터는 컴포저블 Reference, Body에는 `linkTo()`함수를 사용 (Parent는 `ConstraintLayout` 본인)
   <pre><code>
     modifier = Modifier.constrainAs(name) {
                  start.linkTo(profileImage.end, margin = 12.dp)
                  top.linkTo(profileImage.top)
              })
   </code></pre>

### 부가 기능
#### Chain
다중 뷰의 동일한 또는 편향된 간격을 가능하게 한다.
<pre><code>
val (icon1, icon2, icon3) = createRefs()
createHorizontalChain(icon1, icon2, icon3, chainStyle = ChainStyle.SpreadInside)
</code></pre>

#### Barriers
가장 큰 요소를 기준으로 다중 요소의 Anchor Point를 생성한다.<br><br>
예를 들어, 수직 정렬된 2개의 텍스트가 길이가 다르고, 가장 긴 길이의 텍스트에 맞추어 오른쪽에 버튼을 두고 싶을 때,
두 텍스트의 Reference로 EndBarrier를 생성할 수 있다.
<pre><code>
val endBarrier = createEndBarrier(text1, text2)

Button(
    onClick = { },
    modifier = Modifier.constrainAs(button) {
        start.linkTo(endBarrier, margin = 8.dp)
        top.linkTo(parent.top)
    }
)
</code></pre>

#### Guidelines
요소들을 퍼센트 기반 위치에 배치할 수 있도록 도와주는 보조선으로, 실제로 화면에 보이지 않는 가상의 선이다.<br><br>
예를 들어, Start에서 50% 지점에 GuideLine을 생성하면, `ContraintLayout` 내부의 요소를
GuideLine을 사용하여 레이아웃 Start 50% 위치의 오른쪽, 왼쪽에 배치할 수 있다.
<pre><code>
val guideline = createGuidelineFromStart(0.5f)

Text(
    "Hello World",
    modifier = Modifier.constrainAs(text) {
        start.linkTo(guideline)
        top.linkTo(parent.top)
    }
)
</code></pre>

## 참고 자료
- 🧩 [🔥 Mastering ConstraintLayout in Jetpack Compose: From Basics to Advanced](https://proandroiddev.com/mastering-constraintlayout-in-jetpack-compose-from-basics-to-advanced-cbe1cb1a4b2b)


