## suspending 함수 콜백으로 전환하기

Android Weekly #688 **"Suspending functions or flows into callbacks"** 를 읽고 작성하는 글이다.

### suspending 함수 -> 콜백 함수 전환이 필요한 경우?
내부 코루틴 코드를 외부 콜백 환경과 연결하기 위해 필요하다. 몇 가지 예시로는, 
