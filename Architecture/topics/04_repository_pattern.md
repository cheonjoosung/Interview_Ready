### Repository 패턴

#### 개념
데이터 소스(네트워크, DB, 캐시)를 추상화하여 ViewModel/UseCase가 데이터의 출처를 알 필요 없도록 하는 패턴입니다.

#### 역할
```
ViewModel / UseCase
        ↓
   Repository  ← 데이터 출처 결정 로직
    ↙      ↘
Remote API   Local DB (Room)
(Retrofit)   + Cache
```

#### 코드 예시

```kotlin
interface ProductRepository {
    suspend fun getProducts(): Flow<List<Product>>
    suspend fun getProduct(id: String): Result<Product>
    suspend fun refreshProducts()
}

class ProductRepositoryImpl(
    private val api: ProductApi,
    private val dao: ProductDao,
    private val networkMonitor: NetworkMonitor
) : ProductRepository {

    // DB를 Single Source of Truth로 사용
    override suspend fun getProducts(): Flow<List<Product>> =
        dao.getAllProducts().map { it.map(ProductEntity::toDomain) }

    override suspend fun refreshProducts() {
        if (networkMonitor.isOnline()) {
            val remote = api.fetchProducts()
            dao.replaceAll(remote.map(ProductDto::toEntity))
        }
    }

    override suspend fun getProduct(id: String): Result<Product> = runCatching {
        dao.getProduct(id)?.toDomain()
            ?: api.fetchProduct(id).also { dto ->
                dao.insert(dto.toEntity())
            }.toDomain()
    }
}
```

#### Single Source of Truth (SSOT)
- Room DB가 유일한 진실의 소스
- 네트워크 데이터는 DB에 저장 후 DB에서 읽음
- Offline 지원 자동으로 가능

#### 면접 질문
- **Q: Repository 없이 ViewModel에서 Retrofit을 직접 쓰면 안 되나?**
  A: 가능하지만, 데이터 소스가 바뀔 때 ViewModel을 수정해야 하고, 테스트 시 Retrofit 의존성을 제거하기 어렵습니다.

- **Q: Repository와 UseCase가 겹치는 느낌인데, 차이는?**
  A: Repository는 데이터 접근 추상화, UseCase는 비즈니스 로직 표현입니다. 단순 앱은 UseCase 없이 Repository만 써도 됩니다.
