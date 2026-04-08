### Clean Architecture

#### 개념
Uncle Bob(Robert C. Martin)이 제안한 아키텍처로, 관심사를 계층으로 분리하여
각 계층이 안쪽 계층에만 의존하도록 합니다 (의존성 규칙, Dependency Rule).

#### 계층 구조 (바깥 → 안)
```
[Framework & Drivers]  ← Android Framework, Room, Retrofit, UI
        ↓
[Interface Adapters]   ← ViewModel, Repository Impl, Mapper
        ↓
[Application]          ← UseCase (= Interactor)
        ↓
[Domain]               ← Entity, Repository Interface
```

> 의존성은 항상 안쪽(Domain)을 향해야 합니다. Domain은 어떤 외부 라이브러리도 의존하지 않습니다.

#### Android에서의 일반적인 구조
```
app/
├── data/
│   ├── remote/         # API, DTO
│   ├── local/          # Room DAO, Entity
│   └── repository/     # Repository 구현체
├── domain/
│   ├── model/          # Domain Entity (순수 Kotlin)
│   ├── repository/     # Repository 인터페이스
│   └── usecase/        # UseCase
└── presentation/
    ├── viewmodel/      # ViewModel
    └── ui/             # Activity, Fragment, Composable
```

#### UseCase 예시

```kotlin
// Domain Layer - 외부 의존성 없음
data class User(val id: String, val name: String, val email: String)

interface UserRepository {
    suspend fun getUser(id: String): Result<User>
    suspend fun updateUser(user: User): Result<Unit>
}

class GetUserUseCase(private val repository: UserRepository) {
    suspend operator fun invoke(userId: String): Result<User> {
        require(userId.isNotBlank()) { "userId must not be blank" }
        return repository.getUser(userId)
    }
}

// Data Layer - Repository 구현
class UserRepositoryImpl(
    private val api: UserApi,
    private val dao: UserDao
) : UserRepository {
    override suspend fun getUser(id: String): Result<User> = runCatching {
        dao.getUser(id)?.toDomain() ?: api.fetchUser(id).toDomain()
    }
}

// Presentation Layer
class UserViewModel(private val getUser: GetUserUseCase) : ViewModel() {
    fun load(id: String) = viewModelScope.launch {
        getUser(id).onSuccess { /* update state */ }
    }
}
```

#### 장단점

| 장점 | 단점 |
|------|------|
| 테스트 용이성 (Domain 순수 Kotlin) | 초기 보일러플레이트 많음 |
| 각 계층 독립적 변경 가능 | 소규모 프로젝트에 과도함 |
| 비즈니스 로직 재사용 (KMP 등) | 학습 곡선 있음 |

#### 면접 질문
- **Q: UseCase가 반드시 필요한가?**
  A: 단순 CRUD는 ViewModel에서 Repository를 직접 사용해도 됩니다. 복잡한 비즈니스 로직, 여러 Repository 조합, 재사용 필요 시 UseCase를 도입합니다.

- **Q: Data Layer의 DTO와 Domain Model을 왜 분리하나?**
  A: API 응답 형식 변경이 Domain 모델에 영향을 주지 않도록 하기 위해서입니다. Mapper로 변환합니다.
