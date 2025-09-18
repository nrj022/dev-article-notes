## Paging을 사용한 Offline-First 방식

Android Weekly #692의 **"Offline-First or Bust: How Room, WorkManager & Paging 3 Keep Your App Alive Without Internet"** 를 읽고 작성하는 글이다.

### Paging, 페이징 라이브러리
- 대규모 데이터를 로드할 때, 스크롤하여 표시하는 데이터를 효율적으로 관리할 때 주로 사용된다.  
- 페이지 단위로 나누어 데이터를 로드하고, 사용자가 로드한 데이터를 끝까지 스크롤 했을 경우, 자동으로 다음 데이터를 로드한다.
- 따라서, 대규모 데이터를 필요한 만큼 나누어 로드할 때 유용하게 사용되며 시스템 리소스를 효율적으로 사용할 수 있다.

### 시스템 리소스를 절약한다는건?  
채팅을 생각해보자. 만약 그동안 쌓인 채팅 데이터가 10,000개 라면?
- 기본 데이터 전체 로드 방식
  - 스마트폰 메모리: 10,000개의 데이터가 모두 메모리로 올라가며, 심할 경우 메모리 부족으로 앱이 강제 종료될 수 있다.
  - 네트워크 (데이터): 채팅 창에 들어가자마자 10,000개 데이터를 모두 다운로드 한다. 로딩 시간은 길어지고, 모바일 데이터 소모량은 늘어난다.
  - CPU: 서버에서 Json으로 받은 10,000개의 데이터를 한번에 분석하고 객체로 변환하는 과정에 부담을 가진다.   
- Paging Library
  - 대규모 데이터를 20~30개 정도의 작은 양으로 쪼개 로드하기 때문에 메모리, 네트워크, CPU 리소스를 절약할 수 있다.
 
결론적으로, 사용자는 데이터와 배터리를 절약하며 안정적이고 부드러운 사용감을 느낄 수 있다.

### Offline-First 방식은 어떻게 동작하는가?
- 일단 기본적인 Offline-First 방식은 다음과 같은 흐름을 따른다:
  - 서버 데이터 조회 -> Room DB를 사용해 로컬 DB에 데이터 저장 -> 네트워크가 없는 상황에서도 로컬 DB 데이터를 표시
- 여기서 만약 사용자가 요청을 보내고 싶다면?
  - 채팅을 예시로, 네트워크가 연결되지 않은 상태에서 사용자가 채팅을 보낼 경우, 그 내용은 즉시 Room DB에 저장된다.
  - 그리고 여기서 `WorkManager`가 등장한다. 나중에 서버가 연결되면 이 채팅을 전달하도록 `WorkManager`에게 채팅을 예약한다.
- 이런 과정이 사용자에게 어떻게 표시될까?
  - 채팅을 예시로, 서버에 실제로 post되기 전까지 채팅 옆에 전송중 표시를 띄울 수 있다.
  - 실제로 post를 수행한 경우, 성공과 실패에 따라 UI를 다르게 표시하면 된다.
  - 이러한 방식 덕분에 오프라인에서 보낸 채팅을 다른 화면에 다녀온 후에도 '전송 중' 상태 그대로 확인할 수 있는데,
    이는 상태가 Local DB에 저장되어 유지되기 때문에 가능한 것이다. 
   
### Offline-First 아키텍쳐와 Paging의 로컬 DB와 원격 서버 사이의 연결: `RemoteMediator`
- Offline-First 아키텍쳐에서는 데이터를 보여주기 위해 항상 로컬 DB를 먼저 바라본다.
- 로컬 DB에서 더 이상 표시할 데이터가 없는 경우, `RemoteMediator`가 호출되고
  `RemoteMediator`는 다음 페이지를 계산해 서버에 네트워크 통신을 실행한다.
- 서버로부터 받은 데이터는 UI로 직접 전달되지 않고 로컬 DB에 저장한다.
- 저장이 완료되면, `RemoteMediator`는 Paging 라이브러리에 데이터를 다시 로드하라고 알린다.

이러한 과정을 통해, 사용자는 자연스럽게 효율적인 데이터 로딩을 경험할 수 있다.  

![페이징 라이브러리 아키텍쳐](https://developer.android.com/static/topic/libraries/architecture/images/paging3-library-architecture.svg?hl=ko)
_이미지 출처: [안드로이드 공식 문서](https://developer.android.com/topic/libraries/architecture/paging/v3-overview?hl=ko)_

## 출처
- 🧩[Offline-First or Bust: How Room, WorkManager & Paging 3 Keep Your App Alive Without Internet](https://proandroiddev.com/offline-first-or-bust-how-room-workmanager-paging-3-keep-your-app-alive-without-internet-55c65258d138)
- 🧩[Paging 라이브러리 개요](https://developer.android.com/topic/libraries/architecture/paging/v3-overview?hl=ko)
