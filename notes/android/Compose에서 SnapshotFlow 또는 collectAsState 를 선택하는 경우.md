## Compose에서 SnapshotFlow 또는 collectAsState 를 선택하는 경우

### `collectAsState`의 사용
- Flow를 구독하고 상태 변화에 따라 UI의 Recomposition이 자동으로 필요한 경우
- 사용하기 쉽고 ViewModel → UI 상태 전달에 용이
- **하지만,** 작은 변화라도 모든 변화에 Recomposition이 발생하고
- 컴포저블이 시작된 순간부터 예외 없이 수집을 시작하기 때문에
- 스크롤이나 제스쳐 상태를 관찰하기엔 적합하지 않다.

### `snapshotFlow`의 사용
- Compose 상태를 Flow 로 전환, 불필요한 Recomposition 없이 상태 변화에 반응할 수 있음
- `.filter()`, `.distinctUntilChanged()` 등을 사용해 특정 조건에서만 State 변화를 Collect 할 수 있음
- Compose 상태 변화의 Side Effect를 주기에 적절하고 코루틴 내부에서 사용된다.
- **하지만,** UI에서 직접 사용 가능한 형태가 아니다. (필요하다면 collectAsState 등으로 Flow → State로 변환이 필요)

### `collectAsState`와 `snapshotFlow`의 적절한 사용
- `collectAsState`
  - 뷰모델의 Flow나 StateFlow를 구독하는 경우
  - UI에 데이터를 표시하는 경우
  - 사용자가 몰라도 되는 Update가 자주 발생하지 않는 경우
- `snapshotFlow` 
  - Compose의 Scroll, gestures, animations등에 반응하는 경우
  - Recomposition 없이 Side Effect를 발생시키고 싶은 경우

### 결론
- `collectAsState`와 `snapshotFlow`는 상호보완적이다.
- 이 둘을 적절히 사용하면 불필요한 Recomposition을 줄일 수 있고
- 앱의 반응성을 개선할 수 있다.
- 또한, 깔끔하고 예측가능한 코드를 작성할 수 있다.

## 출처
- 🧩 [SnapshotFlow or collectAsState? How to pick the right tool for Jetpack Compose]([https://proandroiddev.com/creating-an-engaging-progress-button-in-jetpack-compose-29ff8d5e383c](https://proandroiddev.com/snapshotflow-or-collectasstate-how-to-pick-the-right-tool-for-jetpack-compose-d6f1cc9d2123))
