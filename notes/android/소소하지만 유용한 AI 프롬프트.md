## 소소하지만 유용한 AI 프롬프트

Android Weekly #699 "Microdosing AI for Mobile Dev"를 읽고 작성하는 글이다.

### Android Gemini 활용방안 (+ 예시 프롬프트)

코드 작성을 위한 활용이 아닌 디버깅, 코드 리뷰, 리소스 정리, 정규식 검색 등 소소하지만 귀찮은 부분을 개발 작업 흐름에 맞게 빠르게 처리해주는 몇가지 AI 활용 방안을 제안한다.
  
1. UI 디버깅 시 해당 화면의 코드 위치를 파악할 때, 스크린샷을 활용해 질문하기 
> Given this screenshot of my app, show me where this is defined and where I can make edits to this UI.

2. PR 리뷰 (팀원 코드 리뷰 전, 빠른 1차 검토가 필요할 때)
> (가독성, 성능, 엣지 케이스, 버그 가능성 점검)  
> Here’s a PR diff. Review this code change in a kind but critical way.
> Highlight anything that looks error prone, consider edge cases, evaluate general performance.
> Favor readability. Summarize into actionable bullet points with must fixes and suggestions.

3. 오래된 프로젝트에서, 안 쓰는 리소스 정리할 때 (Android Studio 의 Unused Resource 도구가 대형 프로젝트에서 실패할 떄 특히 유용)
> Given this resource, does it have any uses? If not can you remove it?

4. 정규식으로 Find in Files
> (하드코딩된 문자열 찾기)  
> Give me a regex for IntelliJ's search to find all Jetpack Compose Text() calls
> where the text parameter is a hardcoded string literal, not a stringResource() call.
>   
> (하드코딩된 색상 찾기)  
> I'm searching my .xml layout files. Write a regex to find any XML attribute that ends in Color (like android:textColor)
> and is set to a hardcoded hex value that starts with #.

### 출처
- 🧩 [Microdosing AI for Mobile Dev](https://blog.mmckenna.me/microdosing-ai-for-mobile-dev)
