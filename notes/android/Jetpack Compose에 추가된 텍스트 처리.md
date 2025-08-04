## Jetpack Compose에 추가된 텍스트 처리

Android Weekly #686의 **Auto-sizing Text in Jetpack Compose with BasicText** 아티클을 읽고 
관련된 내용을 정리한 글이다.

### Compose에서의 텍스트 관리 ?
Jetpack Compose와 같은 선언적인 프레임워크에서는 동적인 텍스트를 잘 처리하는 것이 중요하다.
텍스트의 길이는 계속해서 달라질 수 있기 때문에, 길이가 길어지면 글자 크기를 작게하거나 짧아지면 크게 하는 등,
불필요한 여백이나 텍스트 잘림이 없도록 조절하는 것이 좋다.  

`BasicText` 컴포저블은 **Compose BOM 2025.04.01 버전 이후**부터 이러한 텍스트 처리를 위한 **자동 크기 조절 기능(Auto-resizing)을 제공**한다. 
(우리가 많이 사용하는 `Text`도 `BasicText`를 기반으로 한다.) 

### `BasicText`의 Auto-sizing
텍스트가 너무 많은 경우 & 너무 적은 경우, BasicText의 `autoSize = TextAutoSize.StepBased()`를 사용해 조절할 수 있다.
이 속성은 텍스트의 길이에 따라 단계별로 텍스트의 크기를 조절할 수 있다.
```
BasicText(
    text = "Hello World",
    maxLines = 1,
    autoSize = TextAutoSize.StepBased()
)
```

### 그 외 Jetpack Compose 2025.04 업데이트 (일부)
- 기존 `TextOverflow.Ellipsis`외에 **`TextOverflow.StartEllipsis`와 `TextOverflow.MiddleEllipsis` 옵션이 추가**되었다.  
  이제는 텍스트 길이가 길어졌을 때 텍스트 중간 혹은 시작 부분도 ...으로 생략할 수 있다.
  
- `AnnotatedString` (일부분에 스타일이나 의미를 부여 가능한 문자열)이 **HTML 포맷팅을 지원**한다.  
  `.fromHTML()` 옵션으로 HTML을 텍스트로 사용할 수 있다.
  
- Modifier에 **`animateBounds` 옵션이 추가**되었다.  
  컴포넌트의 크기나 위치가 변경되었을 때 자동으로 이를 **애니메이션 처리**할 수 있다.

## 출처
- 🧩[Auto-sizing Text in Jetpack Compose with BasicText](https://proandroiddev.com/auto-sizing-text-in-jetpack-compose-with-basictext-effbc41502fa)
- 🧩[What’s new in the Jetpack Compose April ’25 release](https://android-developers.googleblog.com/2025/04/whats-new-in-jetpack-compose-april-25.html)
