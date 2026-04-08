### Hilt (의존성 주입)

#### 개념
Dagger2 기반의 Android 공식 DI 라이브러리입니다.
보일러플레이트를 크게 줄여주며 Android 컴포넌트 생명주기와 자동으로 연동됩니다.

#### 설정
```kotlin
// build.gradle (app)
plugins {
    id("com.google.dagger.hilt.android")
    kotlin("kapt")
}

dependencies {
    implementation("com.google.dagger:hilt-android:2.51")
    kapt("com.google.dagger:hilt-android-compiler:2.51")
}
```

#### 기본 사용법

```kotlin
// Application에 @HiltAndroidApp 추가 (필수)
@HiltAndroidApp
class MyApplication : Application()

// 모듈 정의
@Module
@InstallIn(SingletonComponent::class)   // 앱 전체 범위
object NetworkModule {

    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient = OkHttpClient.Builder().build()

    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit =
        Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()

    @Provides
    @Singleton
    fun provideUserApi(retrofit: Retrofit): UserApi = retrofit.create(UserApi::class.java)
}

// 인터페이스 바인딩
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    @Binds
    @Singleton
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository
}

// Activity / Fragment에서 사용
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    // Hilt가 자동으로 주입
}

// ViewModel - @HiltViewModel
@HiltViewModel
class UserViewModel @Inject constructor(
    private val userRepository: UserRepository
) : ViewModel()

// Fragment에서 자동 주입
@AndroidEntryPoint
class UserFragment : Fragment() {
    private val viewModel: UserViewModel by viewModels()

    @Inject lateinit var analytics: AnalyticsService
}
```

#### Hilt 스코프

| 컴포넌트 | 스코프 어노테이션 | 생명주기 |
|----------|------------------|---------|
| SingletonComponent | `@Singleton` | 앱 전체 |
| ActivityRetainedComponent | `@ActivityRetainedScoped` | ViewModel과 동일 |
| ActivityComponent | `@ActivityScoped` | Activity |
| FragmentComponent | `@FragmentScoped` | Fragment |
| ViewModelComponent | `@ViewModelScoped` | ViewModel |

#### Hilt vs Koin 비교

| 항목 | Hilt | Koin |
|------|------|------|
| 타입 | 컴파일 타임 DI | 런타임 DI (Service Locator) |
| 오류 감지 | 컴파일 시 | 런타임 시 |
| 설정 복잡도 | 높음 | 낮음 |
| 성능 | 빠름 | 약간 느림 |
| Google 공식 지원 | O | X |

#### 면접 질문
- **Q: `@Singleton`과 `@ActivityScoped`의 차이는?**
  A: `@Singleton`은 앱 전체에서 하나의 인스턴스를 공유합니다. `@ActivityScoped`는 해당 Activity가 살아있는 동안 하나의 인스턴스를 공유하고, Activity가 소멸되면 새 인스턴스가 생성됩니다.

- **Q: `@Inject constructor`와 `@Provides`의 차이는?**
  A: 내가 만든 클래스는 `@Inject constructor`로 직접 주입 지점을 선언합니다. 외부 라이브러리 클래스나 인터페이스처럼 직접 수정할 수 없는 경우 `@Provides`를 사용합니다.
