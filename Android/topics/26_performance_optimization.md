### 안드로이드 성능 최적화 방법
- 🧵 메인 스레드(UI Thread) 최적화
  + 규칙: 메인 스레드에서 오래 걸리는 작업 금지 (ANR 방지)
  + ❌ 피해야 할 작업
    * 네트워크 요청 (HTTP, 소켓 등)
    * DB I/O (Room, SQLite)
    * 대용량 파일 I/O
    * 복잡한 연산 (JSON 파싱, 이미지 처리 등)
  + ✅ 해결 방법
    * Coroutines (Dispatchers.IO/Main)
    * RxJava, WorkManager
    * Handler/Looper/Executor
  + ```kotlin
    fun fetchData() {
      val url = URL("https://example.com")
      val response = url.readText() // ⚠️ 메인 스레드 차단 → ANR 위험
    }

    fun fetchData() {
      lifecycleScope.launch(Dispatchers.IO) {
        val url = URL("https://example.com")
        val response = url.readText()

        withContext(Dispatchers.Main) {
            textView.text = response
        }
      }
    }
    ``` 

- 🎨 UI / 렌더링 최적화
  + 렌더링 속도 유지 (16ms 이내, 60fps)
  + ✅ 최적화 방법
    * 불필요한 Overdraw 방지 (배경 중첩 줄이기)
      1. Android Studio → Debug GPU Overdraw 확인
    * ConstraintLayout / MotionLayout 적극 활용 → 뷰 계층 단순화
    * RecyclerView → ViewHolder 재활용 / DiffUtil 사용
    * 큰 이미지 → Glide/Picasso/Fresco 같은 라이브러리 활용
    * ViewStub / include / merge 태그로 뷰 성능 개선
    * 애니메이션 → Lottie (JSON 기반), GPU 최적화 활용
  + ```kotlin
    // diffUtil 사용
    class MyAdapter : ListAdapter<User, MyViewHolder>(DiffCallback()) {
      class DiffCallback : DiffUtil.ItemCallback<User>() {
        override fun areItemsTheSame(old: User, new: User) = old.id == new.id
        override fun areContentsTheSame(old: User, new: User) = old == new
      }
    }

    // Overdraw 줄이기 -> merge 태그
    <merge xmlns:android="http://schemas.android.com/apk/res/android">
      <TextView android:id="@+id/title" ... />
      <ImageView android:id="@+id/icon" ... />
    </merge>
    ```  

- 🧠 메모리 최적화
  + ✅ 방법
    * 메모리 누수 방지
      1. static Context 보관 금지
      2. Handler → WeakReference 사용
      3. 리스너/콜백 해제 잊지 않기
      4. Fragment → ViewBinding 해제 (onDestroyView)
    * Bitmap 최적화
      1. inSampleSize 로 다운샘플링
      2. Glide → placeholder / thumbnail / cache 전략
    * LruCache 사용 (메모리 캐싱)
    * LeakCanary 같은 툴로 메모리 누수 탐지
  + ```kotlin
    // Bitmap Downsampling
    val options = BitmapFactory.Options().apply { inSampleSize = 4 }
    val bitmap = BitmapFactory.decodeResource(resources, R.drawable.large_img, options)

    // Leak 방지
    private var _binding: FragmentHomeBinding? = null
    private val binding get() = _binding!!

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
      _binding = FragmentHomeBinding.inflate(inflater, container, false)
      return binding.root
    }

    override fun onDestroyView() {
      super.onDestroyView()
      _binding = null // 메모리 누수 방지
    }
    ``` 

- 🔋 배터리 최적화
  + 배터리는 앱 성능 평판에 직결
  + ✅ 방법
    * 불필요한 WakeLock 사용 금지
    * GPS / 센서 → 필요한 시간에만 등록/해제
    * WorkManager, JobScheduler 사용 (백그라운드 반복 작업 최적화)
    * Foreground Service → 꼭 필요한 경우만 사용
    * 네트워크 요청 → 배치 처리, 압축, 캐싱 활용
  + ```kotlin 
    val constraints = Constraints.Builder()
      .setRequiredNetworkType(NetworkType.UNMETERED)
      .setRequiresCharging(true)
      .build()

    val workRequest = PeriodicWorkRequestBuilder<SyncWorker>(15, TimeUnit.MINUTES)
      .setConstraints(constraints)
      .build()

    WorkManager.getInstance(context).enqueue(workRequest)
    ``` 

- 🌐 네트워크 최적화
  + ✅ 방법
    * Retrofit + OkHttp → Connection Pool / Caching / GZIP 압축
    * JSON 파싱 → Gson보다 Moshi / Kotlin Serialization (빠르고 가벼움)
    * 불필요한 중복 요청 방지 (Debounce/Throttle)
    * 이미지 → CDN 사용, WebP 변환, Cache-Control 헤더 적용
    * 서버와의 주기적 Sync → FCM(푸시) 활용 (폴링 최소화)
  + ```kotlin
    val cacheSize = (10 * 1024 * 1024).toLong()
    val cache = Cache(File(context.cacheDir, "http_cache"), cacheSize)

    val client = OkHttpClient.Builder()
      .cache(cache)
      .addInterceptor { chain ->
          val response = chain.proceed(chain.request())
          response.newBuilder()
              .header("Cache-Control", "public, max-age=60")
              .build()
      }
      .build()
    ``` 

- ⚡ 빌드 & 코드 최적화
  + ✅ 방법
    * ProGuard / R8 → 코드 난독화 + 불필요한 코드 제거 + 사이즈 축소
    * 모듈화 / Dynamic Delivery → 필요한 기능만 설치
    * Reified 타입, Inline 함수 사용 (Kotlin)
    * 불필요한 Reflection / 리플렉션 기반 DI 줄이기 (Hilt, Koin 최적 활용)
  + ```gradle
    minifyEnabled true
    shrinkResources true
    proguardFiles getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro"
    ```
  + ```kotlin
    inline fun <reified T> Gson.fromJson(json: String): T =
      this.fromJson(json, object : TypeToken<T>() {}.type)
    ```

- 면접 관련 질문
  + 앱 성능 최적화에서 가장 중요한 포인트는 무엇인가요?
    * 메인 스레드를 가볍게 유지하는 것 (UI 블록 방지)
    * 네트워크/DB/파일 I/O는 백그라운드에서 처리
  + 메모리 누수를 어떻게 방지할 수 있나요?
    * Context/Activity 참조 관리, WeakReference, 리스너 해제, LeakCanary 활용
  + RecyclerView 성능 최적화 방법은?
    * ViewHolder 패턴, DiffUtil, ListAdapter, setHasFixedSize(true), itemView 재활용


---
