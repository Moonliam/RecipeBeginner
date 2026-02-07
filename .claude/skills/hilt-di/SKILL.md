---
name: hilt-di
description: Hilt 의존성 주입 패턴. ViewModel, Repository, UseCase 주입 및 Module 작성 시 사용.
---

# Hilt 의존성 주입 패턴

## 언제 사용

- ViewModel에 Repository/UseCase 주입 시
- Repository에 DataSource 주입 시
- Module 작성 (Firebase, Room 등) 시
- @Inject, @Binds, @Provides 사용 시

---

## Hilt 기본 설정

### 1. Application 클래스

```kotlin
@HiltAndroidApp
class RecipeBeginnerApplication : Application()
```

### 2. Activity/Fragment

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    // Hilt가 자동으로 의존성 주입
}
```

---

## ViewModel 주입

### ViewModel 작성

```kotlin
@HiltViewModel
class RecipeListViewModel @Inject constructor(
    private val getRecipesUseCase: GetRecipesUseCase,
    private val updateRecipeUseCase: UpdateRecipeUseCase
) : ViewModel() {
    // ViewModel 로직
}
```

### Composable에서 사용

```kotlin
@Composable
fun RecipeListScreen(
    viewModel: RecipeListViewModel = hiltViewModel()
) {
    // ViewModel 사용
}
```

**규칙**:
- `@HiltViewModel` 어노테이션 필수
- 생성자에 `@Inject` 추가
- `hiltViewModel()`로 주입 받음

---

## Repository 패턴

### Repository Interface

```kotlin
interface RecipeRepository {
    suspend fun getRecipes(): Result<List<Recipe>>
    suspend fun getRecipeById(id: String): Result<Recipe>
    suspend fun saveRecipe(recipe: Recipe): Result<Unit>
}
```

### Repository 구현

```kotlin
class RecipeRepositoryImpl @Inject constructor(
    private val firebaseDataSource: FirebaseDataSource,
    private val roomDataSource: RecipeDao
) : RecipeRepository {

    override suspend fun getRecipes(): Result<List<Recipe>> {
        return try {
            // Firebase에서 가져오기
            val recipes = firebaseDataSource.fetchRecipes()
            // 로컬 DB에 캐싱
            roomDataSource.insertAll(recipes)
            Result.success(recipes)
        } catch (e: Exception) {
            // 실패 시 로컬 DB에서 가져오기
            val cachedRecipes = roomDataSource.getAll()
            Result.success(cachedRecipes)
        }
    }
}
```

---

## Module 작성

### @Binds - Interface와 구현 연결

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    @Singleton
    abstract fun bindRecipeRepository(
        impl: RecipeRepositoryImpl
    ): RecipeRepository
}
```

**규칙**:
- `abstract class` 사용
- `@Binds`는 `abstract fun`에만 사용
- Interface와 구현체 1:1 매핑

---

### @Provides - 외부 라이브러리 제공

#### Firebase 제공

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object FirebaseModule {

    @Provides
    @Singleton
    fun provideFirebaseAuth(): FirebaseAuth {
        return Firebase.auth
    }

    @Provides
    @Singleton
    fun provideFirestore(): FirebaseFirestore {
        return Firebase.firestore
    }

    @Provides
    @Singleton
    fun provideFirebaseStorage(): FirebaseStorage {
        return Firebase.storage
    }
}
```

#### Room 제공

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    @Provides
    @Singleton
    fun provideDatabase(
        @ApplicationContext context: Context
    ): RecipeDatabase {
        return Room.databaseBuilder(
            context,
            RecipeDatabase::class.java,
            "recipe_database"
        ).build()
    }

    @Provides
    fun provideRecipeDao(database: RecipeDatabase): RecipeDao {
        return database.recipeDao()
    }
}
```

**규칙**:
- `object` 사용 (static)
- `@Provides` + 구체적 반환 타입
- `@ApplicationContext`로 Context 주입

---

## Scope (생명주기)

### Singleton (앱 전체)

```kotlin
@Provides
@Singleton  // ✅ 앱 전체에서 1개만 생성
fun provideFirebaseAuth(): FirebaseAuth {
    return Firebase.auth
}
```

### ViewModelScoped (ViewModel 생명주기)

```kotlin
@Module
@InstallIn(ViewModelComponent::class)
object ViewModelModule {

    @Provides
    @ViewModelScoped  // ViewModel 생명주기에 맞춰 생성/소멸
    fun provideSomeRepository(): SomeRepository {
        return SomeRepositoryImpl()
    }
}
```

### ActivityScoped / FragmentScoped

```kotlin
@InstallIn(ActivityComponent::class)  // Activity 생명주기
@InstallIn(FragmentComponent::class)  // Fragment 생명주기
```

---

## Qualifier (같은 타입 구분)

### 정의

```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class LocalDataSource

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class RemoteDataSource
```

### 제공

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DataSourceModule {

    @Provides
    @LocalDataSource
    fun provideLocalDataSource(dao: RecipeDao): RecipeDataSource {
        return RoomDataSource(dao)
    }

    @Provides
    @RemoteDataSource
    fun provideRemoteDataSource(firestore: FirebaseFirestore): RecipeDataSource {
        return FirebaseDataSource(firestore)
    }
}
```

### 사용

```kotlin
class RecipeRepository @Inject constructor(
    @LocalDataSource private val localDataSource: RecipeDataSource,
    @RemoteDataSource private val remoteDataSource: RecipeDataSource
) {
    // 두 개의 DataSource를 구분해서 사용
}
```

---

## UseCase 패턴

### UseCase 작성

```kotlin
class GetRecipesUseCase @Inject constructor(
    private val repository: RecipeRepository
) {
    suspend operator fun invoke(): Result<List<Recipe>> {
        return repository.getRecipes()
    }
}
```

### ViewModel에서 사용

```kotlin
@HiltViewModel
class RecipeListViewModel @Inject constructor(
    private val getRecipesUseCase: GetRecipesUseCase
) : ViewModel() {

    fun loadRecipes() {
        viewModelScope.launch {
            getRecipesUseCase()  // invoke() 생략 가능
                .onSuccess { recipes ->
                    // 성공 처리
                }
                .onFailure { error ->
                    // 에러 처리
                }
        }
    }
}
```

**규칙**:
- 복잡한 비즈니스 로직만 UseCase로 분리
- 단순 CRUD는 Repository 직접 사용
- `operator fun invoke()` 사용 (함수처럼 호출 가능)

---

## 테스트에서 Hilt 사용

### ViewModel 테스트

```kotlin
@HiltAndroidTest
class RecipeListViewModelTest {

    @get:Rule
    val hiltRule = HiltAndroidRule(this)

    @Inject
    lateinit var repository: RecipeRepository

    private lateinit var viewModel: RecipeListViewModel

    @Before
    fun setup() {
        hiltRule.inject()
        viewModel = RecipeListViewModel(GetRecipesUseCase(repository))
    }

    @Test
    fun `레시피 로딩 테스트`() = runTest {
        // 테스트 코드
    }
}
```

### Mock Repository 제공

```kotlin
@Module
@TestInstallIn(
    components = [SingletonComponent::class],
    replaces = [RepositoryModule::class]
)
abstract class TestRepositoryModule {

    @Binds
    @Singleton
    abstract fun bindRecipeRepository(
        impl: FakeRecipeRepository
    ): RecipeRepository
}
```

---

## Anti-Pattern (하지 말아야 할 것)

### ❌ 1. Application Context를 직접 주입

```kotlin
// ❌ 나쁜 예
class SomeClass @Inject constructor(
    private val application: Application  // ❌ 직접 주입 금지
)

// ✅ 좋은 예
class SomeClass @Inject constructor(
    @ApplicationContext private val context: Context  // ✅ Qualifier 사용
)
```

### ❌ 2. Singleton에 Activity Context 주입

```kotlin
// ❌ 나쁜 예
@Provides
@Singleton
fun provideSomething(activity: Activity) { }  // ❌ 메모리 누수!

// ✅ 좋은 예
@Provides
@Singleton
fun provideSomething(@ApplicationContext context: Context) { }
```

### ❌ 3. @Inject 생성자 없이 @Binds 사용

```kotlin
// ❌ 나쁜 예
class RecipeRepositoryImpl(  // ❌ @Inject 없음
    private val dataSource: DataSource
) : RecipeRepository

// ✅ 좋은 예
class RecipeRepositoryImpl @Inject constructor(  // ✅ @Inject 추가
    private val dataSource: DataSource
) : RecipeRepository
```

---

## 프로젝트 Module 구조

RecipeBeginner 프로젝트에서 권장하는 Module 구조:

```
:core:data/
└── di/
    ├── DatabaseModule.kt       (Room)
    ├── FirebaseModule.kt       (Firebase)
    └── RepositoryModule.kt     (@Binds)

:feature:auth/
└── di/
    └── AuthModule.kt           (Feature별 의존성)
```

**규칙**:
- `core` 모듈은 공통 의존성 (`@Singleton`)
- `feature` 모듈은 기능별 의존성 (`@ViewModelScoped`)

---

## 참고 자료

- [Hilt Documentation](https://dagger.dev/hilt/)
- [Dependency injection with Hilt](https://developer.android.com/training/dependency-injection/hilt-android)
