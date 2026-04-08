### Jetpack Compose 기초

#### 개념
선언형 UI 프레임워크입니다. UI 상태(State)로부터 UI를 자동으로 빌드합니다.

```
UI = f(State)
```

#### 기존 View 시스템 vs Compose

| 항목 | View 시스템 | Compose |
|------|-------------|---------|
| 패러다임 | 명령형 (how) | 선언형 (what) |
| UI 업데이트 | `textView.text = "..."` | Recomposition 자동 |
| 코드 위치 | XML + Kotlin 분리 | Kotlin 단일 |
| 재사용 | Custom View | Composable 함수 |

#### 기본 개념

```kotlin
// Composable 함수
@Composable
fun Greeting(name: String) {
    Text(text = "Hello, $name!")
}

// 상태 관리
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }   // 상태 기억

    Column(horizontalAlignment = Alignment.CenterHorizontally) {
        Text(text = "Count: $count", style = MaterialTheme.typography.headlineMedium)
        Button(onClick = { count++ }) {
            Text("Increment")
        }
    }
}

// 상태 호이스팅 (State Hoisting)
@Composable
fun CounterScreen() {
    var count by remember { mutableStateOf(0) }
    CounterDisplay(count = count, onIncrement = { count++ })
}

@Composable
fun CounterDisplay(count: Int, onIncrement: () -> Unit) {   // Stateless
    Column {
        Text("$count")
        Button(onClick = onIncrement) { Text("+") }
    }
}
```

#### remember vs rememberSaveable

| | `remember` | `rememberSaveable` |
|-|-----------|---------------------|
| Recomposition | 유지 | 유지 |
| Configuration Change | 초기화 | 유지 (Bundle에 저장) |
| Process Death | 초기화 | 초기화 |

#### 자주 쓰는 컴포저블

```kotlin
// 레이아웃
Column { ... }         // 세로 배치
Row { ... }            // 가로 배치
Box { ... }            // 겹치기
LazyColumn { ... }     // RecyclerView 대체

// LazyColumn 예시
LazyColumn {
    items(userList) { user ->
        UserItem(user = user)
    }
}

// 수정자 (Modifier)
Text(
    text = "Hello",
    modifier = Modifier
        .fillMaxWidth()
        .padding(16.dp)
        .background(Color.LightGray)
        .clickable { /* 클릭 처리 */ }
)
```

#### ViewModel과 연동

```kotlin
@Composable
fun UserScreen(viewModel: UserViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    when (val state = uiState) {
        is UserUiState.Loading -> CircularProgressIndicator()
        is UserUiState.Success -> UserContent(state.user)
        is UserUiState.Error   -> ErrorMessage(state.message)
    }
}
```

#### 면접 질문
- **Q: Recomposition이란?**
  A: State가 변경되면 해당 State를 읽는 Composable 함수가 다시 실행되어 UI를 업데이트하는 과정입니다.

- **Q: `remember`가 없으면 어떻게 되나?**
  A: Recomposition이 일어날 때마다 초기값으로 리셋됩니다. `remember`는 Recomposition 사이에 값을 보존합니다.

- **Q: Compose의 성능 최적화 방법은?**
  A: `key` 파라미터 지정, `derivedStateOf` 사용, `@Stable`/`@Immutable` 어노테이션, lambda를 remember로 캐싱 등.
