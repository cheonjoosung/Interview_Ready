### MVI (Model-View-Intent)

#### 개념
단방향 데이터 흐름(UDF, Unidirectional Data Flow)을 엄격하게 적용한 패턴입니다.
State는 불변(immutable)이며, Intent(사용자 이벤트)로만 변경됩니다.

#### 핵심 개념

| 용어 | 설명 |
|------|------|
| **Intent** | 사용자 액션 또는 시스템 이벤트 (ex: 버튼 클릭, 화면 진입) |
| **State** | UI가 렌더링할 불변 상태 (단일 객체) |
| **SideEffect** | 일회성 이벤트 (ex: 토스트, 화면 이동) |

#### 데이터 흐름
```
View --[Intent]-→ ViewModel --[reduce]-→ State --→ View
                       ↓
                  [SideEffect]
```

#### 코드 예시

```kotlin
// Intent (사용자 이벤트 정의)
sealed class LoginIntent {
    data class EmailChanged(val email: String) : LoginIntent()
    data class PasswordChanged(val password: String) : LoginIntent()
    object LoginClicked : LoginIntent()
}

// State (UI 상태 - 불변)
data class LoginState(
    val email: String = "",
    val password: String = "",
    val isLoading: Boolean = false,
    val isLoginEnabled: Boolean = false,
    val errorMessage: String? = null
)

// SideEffect (일회성 이벤트)
sealed class LoginSideEffect {
    object NavigateToHome : LoginSideEffect()
    data class ShowToast(val message: String) : LoginSideEffect()
}

// ViewModel
class LoginViewModel(private val loginUseCase: LoginUseCase) : ViewModel() {

    private val _state = MutableStateFlow(LoginState())
    val state: StateFlow<LoginState> = _state.asStateFlow()

    private val _sideEffect = Channel<LoginSideEffect>()
    val sideEffect = _sideEffect.receiveAsFlow()

    fun handleIntent(intent: LoginIntent) {
        when (intent) {
            is LoginIntent.EmailChanged    -> reduce { copy(email = intent.email, isLoginEnabled = intent.email.isNotBlank() && password.isNotBlank()) }
            is LoginIntent.PasswordChanged -> reduce { copy(password = intent.password, isLoginEnabled = email.isNotBlank() && intent.password.isNotBlank()) }
            LoginIntent.LoginClicked       -> login()
        }
    }

    private fun login() = viewModelScope.launch {
        reduce { copy(isLoading = true) }
        loginUseCase(state.value.email, state.value.password)
            .onSuccess { _sideEffect.send(LoginSideEffect.NavigateToHome) }
            .onFailure { reduce { copy(isLoading = false, errorMessage = it.message) } }
    }

    private fun reduce(block: LoginState.() -> LoginState) {
        _state.update { it.block() }
    }
}

// View
class LoginFragment : Fragment() {
    private val vm: LoginViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        // State 관찰
        viewLifecycleOwner.lifecycleScope.launch {
            vm.state.collect { render(it) }
        }
        // SideEffect 관찰
        viewLifecycleOwner.lifecycleScope.launch {
            vm.sideEffect.collect { handle(it) }
        }
        // 이벤트 전달
        binding.btnLogin.setOnClickListener {
            vm.handleIntent(LoginIntent.LoginClicked)
        }
    }
}
```

#### MVVM vs MVI

| 항목 | MVVM | MVI |
|------|------|-----|
| 상태 관리 | 여러 LiveData/Flow | 단일 State 객체 |
| 예측 가능성 | 보통 | 높음 (상태가 명확) |
| 복잡도 | 낮음 | 높음 |
| 적합한 상황 | 단순한 화면 | 복잡한 상태를 가진 화면 |

#### 면접 질문
- **Q: MVI에서 SideEffect를 State에 넣지 않는 이유는?**
  A: 토스트 표시 같은 일회성 이벤트를 State에 넣으면 재구독 시 다시 실행되는 문제가 있습니다. Channel이나 SharedFlow로 처리합니다.

- **Q: MVI의 단점은?**
  A: 보일러플레이트가 많고, 단순한 화면에 적용하면 과도한 복잡성이 생깁니다.
