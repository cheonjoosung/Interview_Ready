### Unit Testing (단위 테스트)

#### 개념
외부 의존성 없이 개별 함수/클래스의 동작을 검증합니다.
Android에서는 JUnit4/5 + MockK(또는 Mockito)를 주로 사용합니다.

#### 기본 구조 (AAA 패턴)
```
Arrange → Act → Assert
준비    → 실행 → 검증
```

#### ViewModel 테스트

```kotlin
// 의존성
testImplementation("junit:junit:4.13.2")
testImplementation("io.mockk:mockk:1.13.10")
testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.8.0")
testImplementation("app.cash.turbine:turbine:1.1.0")   // Flow 테스트

class UserViewModelTest {

    @get:Rule
    val mainDispatcherRule = MainDispatcherRule()   // 코루틴 Dispatcher 교체

    private val userRepository: UserRepository = mockk()
    private lateinit var viewModel: UserViewModel

    @Before
    fun setUp() {
        viewModel = UserViewModel(userRepository)
    }

    @Test
    fun `loadUser 성공 시 Success 상태가 된다`() = runTest {
        // Arrange
        val user = User(id = "1", name = "홍길동")
        coEvery { userRepository.getUser("1") } returns Result.success(user)

        // Act
        viewModel.loadUser("1")

        // Assert
        viewModel.uiState.test {
            val state = awaitItem()
            assertIs<UserUiState.Success>(state)
            assertEquals("홍길동", state.user.name)
        }
    }

    @Test
    fun `loadUser 실패 시 Error 상태가 된다`() = runTest {
        coEvery { userRepository.getUser(any()) } returns Result.failure(Exception("Network Error"))

        viewModel.loadUser("1")

        viewModel.uiState.test {
            assertIs<UserUiState.Error>(awaitItem())
        }
    }
}

// MainDispatcherRule - 코루틴 테스트용
class MainDispatcherRule(
    val dispatcher: TestCoroutineDispatcher = TestCoroutineDispatcher()
) : TestWatcher() {
    override fun starting(description: Description) {
        Dispatchers.setMain(dispatcher)
    }
    override fun finished(description: Description) {
        Dispatchers.resetMain()
        dispatcher.cleanupTestCoroutines()
    }
}
```

#### Repository 테스트 (Fake 사용)

```kotlin
// Fake Repository (MockK 대신 직접 구현)
class FakeUserRepository : UserRepository {
    private val users = mutableMapOf<String, User>()
    var shouldThrow = false

    override suspend fun getUser(id: String): Result<User> {
        if (shouldThrow) return Result.failure(Exception("Error"))
        return users[id]?.let { Result.success(it) }
            ?: Result.failure(Exception("Not found"))
    }

    fun addUser(user: User) { users[user.id] = user }
}

class UserViewModelTest {
    private val repository = FakeUserRepository()
    private val viewModel = UserViewModel(repository)

    @Test
    fun `사용자가 있으면 Success를 반환한다`() = runTest {
        repository.addUser(User("1", "홍길동"))
        viewModel.loadUser("1")
        // assert...
    }
}
```

#### UseCase 테스트

```kotlin
class GetUserUseCaseTest {
    private val repository: UserRepository = mockk()
    private val useCase = GetUserUseCase(repository)

    @Test
    fun `빈 userId는 IllegalArgumentException을 던진다`() {
        assertThrows<IllegalArgumentException> {
            runBlocking { useCase("") }
        }
    }
}
```

#### 면접 질문
- **Q: Mock과 Fake의 차이는?**
  A: Mock은 동작을 설정하는 라이브러리 객체(MockK, Mockito), Fake는 실제 동작을 흉내내는 테스트용 구현체입니다. Fake가 더 실제와 가깝고 유지보수하기 쉽습니다.

- **Q: `runTest`를 쓰는 이유는?**
  A: `delay()`를 실제로 기다리지 않고 즉시 진행(virtual time)하여 테스트 속도를 높입니다.
