### ViewModel & LiveData

#### ViewModel

##### 역할
- UI 관련 데이터를 저장하고 관리
- Configuration Change(화면 회전 등)에서 데이터 유지
- View의 생명주기와 독립적으로 동작

##### 생명주기
```
Activity Created → ViewModel Created
Activity Rotated → ViewModel Survives (동일 인스턴스)
Activity Finished → ViewModel.onCleared() 호출 → 소멸
```

##### 생성 방법

```kotlin
// 1. by viewModels() (Fragment KTX)
class MyFragment : Fragment() {
    private val vm: MyViewModel by viewModels()
}

// 2. Activity에서 Fragment와 공유
class MyFragment : Fragment() {
    private val vm: SharedViewModel by activityViewModels()
}

// 3. 팩토리 사용 (파라미터가 있을 때)
class MyViewModel(private val id: String) : ViewModel() { ... }

class MyViewModelFactory(private val id: String) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return MyViewModel(id) as T
    }
}

val vm: MyViewModel by viewModels { MyViewModelFactory("123") }
```

##### viewModelScope
```kotlin
class MyViewModel : ViewModel() {
    fun loadData() {
        viewModelScope.launch {   // ViewModel이 소멸하면 자동으로 취소됨
            val data = repository.getData()
            _uiState.value = data
        }
    }
}
```

#### LiveData

##### 특징
- Lifecycle-aware: 활성 상태(STARTED/RESUMED)의 Observer에만 데이터 전달
- 메인 스레드에서 관찰
- 자동 구독 해제 (메모리 누수 방지)

```kotlin
class UserViewModel : ViewModel() {
    private val _user = MutableLiveData<User>()
    val user: LiveData<User> = _user   // 외부에는 읽기 전용 노출

    fun loadUser(id: String) {
        viewModelScope.launch {
            _user.value = repository.getUser(id)    // 메인 스레드
            // _user.postValue(user)                // 백그라운드 스레드
        }
    }
}

// Fragment에서 관찰
viewModel.user.observe(viewLifecycleOwner) { user ->
    binding.textName.text = user.name
}
```

##### LiveData 변환
```kotlin
val userName: LiveData<String> = user.map { it.name }
val combinedData: LiveData<String> = MediatorLiveData<String>().apply {
    addSource(liveData1) { value = "from 1: $it" }
    addSource(liveData2) { value = "from 2: $it" }
}
```

#### LiveData vs StateFlow 비교

| 항목 | LiveData | StateFlow |
|------|----------|-----------|
| 초기값 | 없어도 됨 | 필수 |
| null 허용 | O | X (non-null) |
| 코루틴 연계 | 어색함 | 자연스러움 |
| Lifecycle 인식 | 자동 | 수동 (repeatOnLifecycle) |
| Cold/Hot | Cold | Hot |
| 권장 여부 | 레거시 | 신규 프로젝트 권장 |

#### 면접 질문
- **Q: ViewModel이 화면 회전에서 살아남는 내부 원리는?**
  A: `ViewModelStore`가 `Activity.onRetainNonConfigurationInstance()`를 통해 저장됩니다. 시스템이 새 Activity를 만들 때 `NonConfigurationInstance`에서 `ViewModelStore`를 복구합니다.

- **Q: LiveData를 Fragment에서 관찰할 때 viewLifecycleOwner를 써야 하는 이유는?**
  A: Fragment의 `this`(Fragment 자체)는 backstack에 있을 때도 살아있지만 View는 소멸합니다. `viewLifecycleOwner`를 쓰면 View가 소멸될 때 자동으로 구독이 해제됩니다.
