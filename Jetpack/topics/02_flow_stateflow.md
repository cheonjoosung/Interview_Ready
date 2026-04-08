### Flow & StateFlow & SharedFlow

#### Kotlin Flow 기초

Flow는 비동기적으로 여러 값을 순서대로 내보내는 Cold Stream입니다.

```kotlin
// Flow 생성
fun numbersFlow(): Flow<Int> = flow {
    for (i in 1..5) {
        delay(100)
        emit(i)
    }
}

// 수집 (terminal operator)
viewModelScope.launch {
    numbersFlow()
        .filter { it % 2 == 0 }
        .map { it * it }
        .collect { println(it) }   // 4, 16
}
```

##### 주요 연산자

| 연산자 | 설명 |
|--------|------|
| `map` | 각 값 변환 |
| `filter` | 조건에 맞는 값만 통과 |
| `flatMapLatest` | 새 값이 오면 이전 Flow 취소 후 새 Flow 시작 |
| `combine` | 두 Flow의 최신값 조합 |
| `zip` | 두 Flow를 쌍으로 묶음 |
| `debounce` | 마지막 값 이후 일정 시간 후 emit |
| `distinctUntilChanged` | 이전 값과 같으면 무시 |

#### StateFlow

```kotlin
class CounterViewModel : ViewModel() {
    private val _count = MutableStateFlow(0)   // 초기값 필수
    val count: StateFlow<Int> = _count.asStateFlow()

    fun increment() { _count.update { it + 1 } }
    fun decrement() { _count.update { it - 1 } }
}

// Fragment에서 안전하게 수집
viewLifecycleOwner.lifecycleScope.launch {
    viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.count.collect { count ->
            binding.textCount.text = count.toString()
        }
    }
}
```

> ❗ `launchIn(lifecycleScope)` 대신 `repeatOnLifecycle`을 사용해야 백그라운드에서 수집이 중단됩니다.

#### SharedFlow

일회성 이벤트(토스트, 네비게이션)에 적합한 Hot Stream입니다.

```kotlin
class MyViewModel : ViewModel() {
    private val _events = MutableSharedFlow<UiEvent>()
    val events = _events.asSharedFlow()

    fun showToast(message: String) {
        viewModelScope.launch {
            _events.emit(UiEvent.ShowToast(message))
        }
    }
}

sealed class UiEvent {
    data class ShowToast(val message: String) : UiEvent()
    object NavigateBack : UiEvent()
}
```

#### Cold vs Hot Stream

| | Cold (Flow) | Hot (StateFlow, SharedFlow) |
|-|-------------|------------------------------|
| 수집자 없을 때 | 동작 안 함 | 독립적으로 동작 |
| 수집자마다 | 새 스트림 시작 | 동일 스트림 공유 |
| 현재 상태 | 없음 | StateFlow는 마지막 값 보유 |

#### StateFlow vs SharedFlow 비교

| 항목 | StateFlow | SharedFlow |
|------|-----------|------------|
| 초기값 | 필수 | 없어도 됨 |
| replay | 마지막 1개 기본 | 설정 가능 |
| 중복 emit | 같은 값이면 무시 | 항상 emit |
| 적합한 용도 | UI 상태 | 일회성 이벤트 |

#### 면접 질문
- **Q: `lifecycleScope.launch { flow.collect {} }`의 문제점은?**
  A: 앱이 백그라운드로 가도 수집이 계속되어 불필요한 리소스를 소비합니다. `repeatOnLifecycle(STARTED)`를 사용해야 합니다.

- **Q: `flatMapLatest`와 `flatMapMerge`의 차이는?**
  A: `flatMapLatest`는 새 값이 오면 이전 Flow를 취소합니다(검색 자동완성에 적합). `flatMapMerge`는 모든 Flow를 동시에 수집합니다.
