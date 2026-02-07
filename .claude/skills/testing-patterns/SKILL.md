---
name: testing-patterns
description: 안드로이드 단위 테스트 작성 패턴 (JUnit, MockK, Turbine). ViewModel, Repository, UseCase 테스트 시 사용.
---

# 안드로이드 테스트 작성 패턴

## 언제 사용

- ViewModel 테스트 작성 시
- Repository 테스트 작성 시
- UseCase 테스트 작성 시
- Flow, StateFlow 테스트 시
- Mock 객체 생성 시

---

## 테스트 구조

### AAA 패턴 (Arrange-Act-Assert)

```kotlin
@Test
fun `레시피 로딩 성공 시 UI State 업데이트`() = runTest {
    // Arrange (준비)
    val expectedRecipes = listOf(mockRecipe1, mockRecipe2)
    coEvery { repository.getRecipes() } returns Result.success(expectedRecipes)

    // Act (실행)
    viewModel.loadRecipes()
    advanceUntilIdle()

    // Assert (검증)
    val uiState = viewModel.uiState.value
    assertFalse(uiState.isLoading)
    assertEquals(expectedRecipes, uiState.recipes)
    assertNull(uiState.error)
}
```

---

## MockK 기본 사용법

### Mock 객체 생성

```kotlin
@Test
fun exampleTest() {
    // Mock 생성
    val repository = mockk<RecipeRepository>()

    // Mock 동작 정의
    coEvery { repository.getRecipes() } returns Result.success(emptyList())

    // 테스트 실행
    // ...

    // 호출 검증
    coVerify { repository.getRecipes() }
}
```

### Relaxed Mock

```kotlin
// 모든 함수가 기본값 반환 (Unit, 0, null, emptyList 등)
val repository = mockk<RecipeRepository>(relaxed = true)
```

---

## ViewModel 테스트

### 기본 템플릿

```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
class RecipeListViewModelTest {

    // MainDispatcher를 Test Dispatcher로 교체
    @get:Rule
    val mainDispatcherRule = MainDispatcherRule()

    // Mocks
    private val repository = mockk<RecipeRepository>()
    private val getRecipesUseCase = GetRecipesUseCase(repository)

    private lateinit var viewModel: RecipeListViewModel

    @Before
    fun setup() {
        viewModel = RecipeListViewModel(getRecipesUseCase)
    }

    @After
    fun tearDown() {
        unmockkAll()
    }

    @Test
    fun `초기 상태는 로딩 중이 아님`() {
        // Given & When
        val uiState = viewModel.uiState.value

        // Then
        assertFalse(uiState.isLoading)
        assertTrue(uiState.recipes.isEmpty())
        assertNull(uiState.error)
    }

    @Test
    fun `레시피 로딩 성공`() = runTest {
        // Given
        val recipes = listOf(
            Recipe(id = "1", title = "김치찌개"),
            Recipe(id = "2", title = "된장찌개")
        )
        coEvery { repository.getRecipes() } returns Result.success(recipes)

        // When
        viewModel.loadRecipes()
        advanceUntilIdle()  // 모든 Coroutine 완료 대기

        // Then
        val uiState = viewModel.uiState.value
        assertFalse(uiState.isLoading)
        assertEquals(2, uiState.recipes.size)
        assertNull(uiState.error)
    }

    @Test
    fun `레시피 로딩 실패`() = runTest {
        // Given
        val errorMessage = "네트워크 오류"
        coEvery { repository.getRecipes() } returns Result.failure(Exception(errorMessage))

        // When
        viewModel.loadRecipes()
        advanceUntilIdle()

        // Then
        val uiState = viewModel.uiState.value
        assertFalse(uiState.isLoading)
        assertTrue(uiState.recipes.isEmpty())
        assertEquals(errorMessage, uiState.error)
    }
}
```

---

## MainDispatcherRule (필수)

```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
class MainDispatcherRule : TestWatcher() {
    private val testDispatcher = StandardTestDispatcher()

    override fun starting(description: Description) {
        Dispatchers.setMain(testDispatcher)
    }

    override fun finished(description: Description) {
        Dispatchers.resetMain()
    }
}
```

**위치**: `src/test/kotlin/util/MainDispatcherRule.kt`

**용도**: MainDispatcher를 Test Dispatcher로 교체 (Coroutine 테스트 필수)

---

## Flow 테스트 (Turbine)

### 기본 사용법

```kotlin
@Test
fun `레시피 로딩 중 State 변화`() = runTest {
    // Given
    val recipes = listOf(mockRecipe)
    coEvery { repository.getRecipes() } coAnswers {
        delay(100)  // 지연 시뮬레이션
        Result.success(recipes)
    }

    // When & Then
    viewModel.uiState.test {
        // 초기 상태
        val initialState = awaitItem()
        assertFalse(initialState.isLoading)

        // 로딩 시작
        viewModel.loadRecipes()
        val loadingState = awaitItem()
        assertTrue(loadingState.isLoading)

        // 로딩 완료
        val successState = awaitItem()
        assertFalse(successState.isLoading)
        assertEquals(1, successState.recipes.size)
    }
}
```

### Skip & ExpectMostRecentItem

```kotlin
@Test
fun `마지막 상태만 확인`() = runTest {
    viewModel.uiState.test {
        skipItems(2)  // 처음 2개 상태 건너뛰기

        viewModel.loadRecipes()

        val finalState = expectMostRecentItem()  // 가장 최근 상태
        assertEquals(2, finalState.recipes.size)
    }
}
```

---

## Repository 테스트

### Firebase Mock

```kotlin
class RecipeRepositoryTest {

    private val firestore = mockk<FirebaseFirestore>()
    private val collection = mockk<CollectionReference>()
    private val query = mockk<Query>()

    private lateinit var repository: RecipeRepositoryImpl

    @Before
    fun setup() {
        every { firestore.collection("recipes") } returns collection
        repository = RecipeRepositoryImpl(firestore)
    }

    @Test
    fun `Firestore에서 레시피 가져오기 성공`() = runTest {
        // Given
        val snapshot = mockk<QuerySnapshot>()
        val documents = listOf(mockRecipeDocument)

        every { snapshot.documents } returns documents
        coEvery { collection.get().await() } returns snapshot

        // When
        val result = repository.getRecipes()

        // Then
        assertTrue(result.isSuccess)
        assertEquals(1, result.getOrNull()?.size)
    }
}
```

### Room Mock

```kotlin
class RecipeLocalDataSourceTest {

    private val dao = mockk<RecipeDao>()
    private lateinit var dataSource: RecipeLocalDataSource

    @Before
    fun setup() {
        dataSource = RecipeLocalDataSource(dao)
    }

    @Test
    fun `로컬 DB에서 레시피 가져오기`() = runTest {
        // Given
        val entities = listOf(mockRecipeEntity)
        coEvery { dao.getAll() } returns entities

        // When
        val result = dataSource.getRecipes()

        // Then
        assertEquals(1, result.size)
        coVerify { dao.getAll() }
    }
}
```

---

## UseCase 테스트

```kotlin
class GetRecipesUseCaseTest {

    private val repository = mockk<RecipeRepository>()
    private lateinit var useCase: GetRecipesUseCase

    @Before
    fun setup() {
        useCase = GetRecipesUseCase(repository)
    }

    @Test
    fun `Repository 성공 시 UseCase도 성공`() = runTest {
        // Given
        val recipes = listOf(mockRecipe)
        coEvery { repository.getRecipes() } returns Result.success(recipes)

        // When
        val result = useCase()

        // Then
        assertTrue(result.isSuccess)
        assertEquals(recipes, result.getOrNull())
    }

    @Test
    fun `Repository 실패 시 UseCase도 실패`() = runTest {
        // Given
        coEvery { repository.getRecipes() } returns Result.failure(Exception("Error"))

        // When
        val result = useCase()

        // Then
        assertTrue(result.isFailure)
    }
}
```

---

## Mock 데이터 Factory

### PreviewData.kt (Mock 데이터)

```kotlin
object MockData {
    val mockRecipe1 = Recipe(
        id = "1",
        title = "김치찌개",
        cookingTime = 30,
        difficulty = Difficulty.EASY,
        ingredients = listOf("김치", "돼지고기", "두부"),
        steps = listOf("1단계", "2단계", "3단계")
    )

    val mockRecipe2 = Recipe(
        id = "2",
        title = "된장찌개",
        cookingTime = 20,
        difficulty = Difficulty.EASY,
        ingredients = listOf("된장", "두부", "감자"),
        steps = listOf("1단계", "2단계")
    )

    val mockRecipes = listOf(mockRecipe1, mockRecipe2)
}
```

**위치**: `src/test/kotlin/util/MockData.kt` 또는 각 모듈의 `test` 디렉토리

---

## 테스트 네이밍 컨벤션

### 한국어 네이밍 (권장)

```kotlin
@Test
fun `레시피 로딩 성공 시 UI State 업데이트`() { }

@Test
fun `레시피가 비어있을 때 빈 화면 표시`() { }

@Test
fun `네트워크 오류 시 에러 메시지 표시`() { }
```

### 영어 네이밍 (대안)

```kotlin
@Test
fun `when recipe loading succeeds then update UI state`() { }
```

---

## 자주 사용하는 Assert

```kotlin
// 동등성
assertEquals(expected, actual)
assertNotEquals(notExpected, actual)

// Boolean
assertTrue(condition)
assertFalse(condition)

// Null
assertNull(value)
assertNotNull(value)

// Exception
assertThrows<IllegalArgumentException> {
    // 예외가 발생해야 하는 코드
}

// Collection
assertTrue(list.isEmpty())
assertTrue(list.contains(item))
assertEquals(2, list.size)
```

---

## Verify (호출 검증)

```kotlin
// 1번 호출 확인
coVerify { repository.getRecipes() }
verify { someFunction() }

// N번 호출 확인
coVerify(exactly = 2) { repository.getRecipes() }

// 호출되지 않았는지 확인
coVerify(exactly = 0) { repository.deleteRecipe(any()) }

// 순서 확인
coVerifyOrder {
    repository.getRecipes()
    repository.saveRecipe(any())
}
```

---

## Anti-Pattern (하지 말아야 할 것)

### ❌ 1. advanceUntilIdle() 없이 비동기 테스트

```kotlin
// ❌ 나쁜 예
@Test
fun test() = runTest {
    viewModel.loadRecipes()
    val uiState = viewModel.uiState.value  // ❌ 아직 로딩 중일 수 있음
    assertEquals(2, uiState.recipes.size)
}

// ✅ 좋은 예
@Test
fun test() = runTest {
    viewModel.loadRecipes()
    advanceUntilIdle()  // ✅ 모든 Coroutine 완료 대기
    val uiState = viewModel.uiState.value
    assertEquals(2, uiState.recipes.size)
}
```

### ❌ 2. MainDispatcherRule 없이 ViewModel 테스트

```kotlin
// ❌ 나쁜 예
class ViewModelTest {
    // MainDispatcherRule 없음
}

// ✅ 좋은 예
class ViewModelTest {
    @get:Rule
    val mainDispatcherRule = MainDispatcherRule()
}
```

### ❌ 3. unmockkAll() 없이 Mock 재사용

```kotlin
// ❌ 나쁜 예
@After
fun tearDown() {
    // unmockkAll() 없음 - 다른 테스트에 영향
}

// ✅ 좋은 예
@After
fun tearDown() {
    unmockkAll()
}
```

---

## 참고 자료

- [MockK Documentation](https://mockk.io/)
- [Turbine (Flow Testing)](https://github.com/cashapp/turbine)
- [Testing Coroutines](https://developer.android.com/kotlin/coroutines/test)
