### Room Database

#### 개념
SQLite를 추상화한 Android의 공식 ORM 라이브러리입니다.
컴파일 타임에 SQL 쿼리를 검증합니다.

#### 구성 요소

| 컴포넌트 | 역할 | 어노테이션 |
|----------|------|-----------|
| Entity | 테이블 정의 | `@Entity` |
| DAO | 데이터 접근 메서드 | `@Dao` |
| Database | DB 인스턴스 | `@Database` |

#### 코드 예시

```kotlin
// Entity
@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey val id: String,
    @ColumnInfo(name = "user_name") val name: String,
    val email: String,
    @ColumnInfo(defaultValue = "CURRENT_TIMESTAMP") val createdAt: String = ""
)

// DAO
@Dao
interface UserDao {
    @Query("SELECT * FROM users WHERE id = :id")
    suspend fun getUser(id: String): UserEntity?

    @Query("SELECT * FROM users ORDER BY user_name ASC")
    fun getAllUsers(): Flow<List<UserEntity>>   // Flow로 실시간 감지

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUser(user: UserEntity)

    @Update
    suspend fun updateUser(user: UserEntity)

    @Delete
    suspend fun deleteUser(user: UserEntity)

    @Query("DELETE FROM users")
    suspend fun deleteAll()
}

// Database
@Database(entities = [UserEntity::class], version = 1, exportSchema = true)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao

    companion object {
        @Volatile private var instance: AppDatabase? = null

        fun getInstance(context: Context): AppDatabase =
            instance ?: synchronized(this) {
                instance ?: Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "app_database"
                ).build().also { instance = it }
            }
    }
}
```

#### 마이그레이션
```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("ALTER TABLE users ADD COLUMN phone TEXT NOT NULL DEFAULT ''")
    }
}

Room.databaseBuilder(context, AppDatabase::class.java, "app_db")
    .addMigrations(MIGRATION_1_2)
    .build()
```

#### TypeConverter (복잡한 타입 저장)
```kotlin
class DateConverter {
    @TypeConverter
    fun fromTimestamp(value: Long?): Date? = value?.let { Date(it) }

    @TypeConverter
    fun dateToTimestamp(date: Date?): Long? = date?.time
}

@Database(...)
@TypeConverters(DateConverter::class)
abstract class AppDatabase : RoomDatabase()
```

#### 면접 질문
- **Q: Room에서 suspend 함수와 Flow의 차이는?**
  A: `suspend`는 일회성 쿼리에 사용합니다. `Flow`는 데이터 변경 시 자동으로 새 값을 emit합니다(실시간 감지).

- **Q: `@Insert(onConflict = OnConflictStrategy.REPLACE)`는 어떻게 동작하나?**
  A: Primary Key가 충돌하면 기존 행을 삭제하고 새 행을 삽입합니다. 주의: 연관된 Foreign Key 데이터도 삭제될 수 있습니다.

- **Q: Room DB를 싱글턴으로 만드는 이유는?**
  A: DB 파일 접근은 비용이 크고, 여러 인스턴스가 동시에 접근하면 데이터 일관성 문제가 생깁니다.
