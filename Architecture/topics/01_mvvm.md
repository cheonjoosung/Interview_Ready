### MVVM (Model-View-ViewModel)

#### 개념
MVVM은 UI 로직과 비즈니스 로직을 분리하는 아키텍처 패턴입니다.
Android에서는 Jetpack ViewModel + LiveData/StateFlow와 함께 사용하는 것이 표준입니다.

#### 구성 요소

| 레이어 | 역할 | Android 구현체 |
|--------|------|----------------|
| **Model** | 데이터 & 비즈니스 로직 | Repository, Room, Retrofit |
| **View** | UI 표시, 사용자 입력 처리 | Activity, Fragment, Composable |
| **ViewModel** | View에 노출할 상태 관리 | `androidx.lifecycle.ViewModel` |

#### 데이터 흐름
```
View  →(이벤트)→  ViewModel  →(요청)→  Model(Repository)
View  ←(상태)←   ViewModel  ←(데이터)← Model(Repository)
```

#### 코드 예시

```kotlin
// Model - Repository
class UserRepository(private val api: UserApi, private val db: UserDao) {
    suspend fun getUser(id: String): Result<User> = runCatching {
        db.getUser(id) ?: api.fetchUser(id).also { db.insertUser(it) }
    }
}

// ViewModel
class UserViewModel(private val repository: UserRepository) : ViewModel() {

    private val _uiState = MutableStateFlow<UserUiState>(UserUiState.Loading)
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()

    fun loadUser(id: String) {
        viewModelScope.launch {
            _uiState.value = UserUiState.Loading
            repository.getUser(id)
                .onSuccess { _uiState.value = UserUiState.Success(it) }
                .onFailure { _uiState.value = UserUiState.Error(it.message) }
        }
    }
}

sealed class UserUiState {
    object Loading : UserUiState()
    data class Success(val user: User) : UserUiState()
    data class Error(val message: String?) : UserUiState()
}

// View (Fragment)
class UserFragment : Fragment() {
    private val viewModel: UserViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        viewLifecycleOwner.lifecycleScope.launch {
            viewModel.uiState.collect { state ->
                when (state) {
                    is UserUiState.Loading -> showLoading()
                    is UserUiState.Success -> showUser(state.user)
                    is UserUiState.Error   -> showError(state.message)
                }
            }
        }
        viewModel.loadUser("user_id")
    }
}
```

#### MVVM vs MVP 비교

| 항목 | MVVM | MVP |
|------|------|-----|
| View-Presenter 결합 | 약함 (데이터 바인딩/옵저버) | 강함 (인터페이스 직접 참조) |
| 테스트 용이성 | 높음 | 보통 |
| 보일러플레이트 | 적음 | 많음 |
| Android 공식 지원 | O (Jetpack) | X |

#### 면접 질문
- **Q: ViewModel이 Configuration Change에서 살아남는 이유는?**
  A: ViewModelStore가 Activity의 NonConfigurationInstance로 저장되므로, 회전 등 설정 변경 시에도 유지됩니다.

- **Q: ViewModel에서 View 참조를 가지면 안 되는 이유는?**
  A: ViewModel의 생명주기가 View보다 길어, View가 destroy된 후에도 ViewModel이 살아있어 메모리 누수가 발생합니다.

- **Q: LiveData vs StateFlow 차이는?**
  A: LiveData는 Lifecycle-aware지만 초기값이 없고 Cold입니다. StateFlow는 초기값이 필수이고 Hot이며, 코루틴 친화적입니다.
