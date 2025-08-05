## 스케일이 큰 앱에서 자주 발생하는 Compose Mistakes

Android Weekly #686의 **"Top 10 Compose Mistakes in Large-Scale Apps"** 를 읽고 정리한 글이다.
지금 진행하는 프로젝트에 반영할 부분을 적어보려고 한다. 차근히 고쳐나가야될 것 같다.

### 주요 Compose Mistakes

**1. 모듈화를 무시하는 것**
   
   - 확장성과 가독성이 떨어지고, 이후 유지관리가 어려워진다.
   - 역할에 따라 모듈로 분리하자 (이 부분에 대해서는 좀 더 고민이 필요할 것 같다. 정확히 무엇을 기준으로 모듈을 분리해야하나?)
   - 모듈화의 이점은 확장성과 캡슐화, 테스트 가능성과 빌드 시간에 있다.

**2. 비효율적인 List Handling** 

    ```
    // Before
    LazyColumn {
        items(itemsList) { item ->
            ItemComposable(item)
        }
    }
  
    // After
    LazyColumn {
        items(itemsList, key = { it.id }) { item ->
            ItemComposable(item)
        }
    }
    ``` 

   두 코드의 차이점은? 바로 Key다.  
   Key를 사용하면 어떤 점이 좋은가? key를 통해 상태를 기억할 수 있다. 각 key의 요소를 이전과 비교해서,
   변화가 없으면 Recomposition이 일어나지 않는다는 것이다. 결론적으로 **불필요한 재구성을 줄일 수 있다.** 
    
   또한 이런 불필요한 재구성이 줄어들면, `LazyColumn`이 좀 더 효율적으로 아이템을 관리해 스크롤 성능이 향상된다고 한다.

**3. 테마와 디자인 시스템을 과소 평가하는 것**

   일관성을 유지하고 관리하기 쉽도록 하려면 중앙 집중식이 필수적이다.
   색상의 경우 다크모드를 쉽게 설정할 수도 있고 색상 변경에도 용이하다.

   현재 프로젝트에서는 텍스트는 중앙집중식으로 관리하지만
   색상의 경우 ColorScheme를 만들었지만, 사용되는 색상이 많고 바로바로 파악하기가 어려워서
   명시적인 색상 이름으로 사용하고 있었는데, 이 부분에 대해서는 어떤 방식이 더 나을지 고민이 더 필요할 것 같다.

**4. 접근성을 무시하는 것**

   지금 프로젝트에서는 `ContentDecription`으로 어느정도 접근성을 고려하고 있는데,
   좀 더 명확하게 의미를 전달하고 있는지 더 고민해보면 좋을 것 같다.
   ```
   Text(
      text = "Button",
      modifier = Modifier.semantics { contentDescription = "Submit Button" }
   )
   ```
**5. UI Test를 작성하지 않는 것**

   Junit Test 에 대해 아직 지식이 부족해서 `@Preview`로 시각적으로만 확인해왔는데, 
   확실히 상호작용에 따른 결과를 테스트하기 위해서는 자동화된 테스트를 더 이상 미루지 말아야할 것 같다.

## 출처
- 🧩[Top 10 Compose Mistakes in Large-Scale Apps](https://proandroiddev.com/top-10-compose-mistakes-in-large-scale-apps-55990e1b20ee)
- 🧩[Android 앱 모듈화 가이드](https://developer.android.com/topic/modularization?hl=ko)
