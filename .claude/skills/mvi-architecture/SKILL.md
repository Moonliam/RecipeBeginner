---
name: mvi-architecture
description: MVI (Model-View-Intent) 아키텍처 패턴. ViewModel, UI State, Event 설계 및 단방향 데이터 플로우 구현 시 사용.
---

# MVI 아키텍처 패턴

## 언제 사용

- ViewModel 작성 시
- UI State 정의 시
- UI Event 처리 시
- 단방향 데이터 플로우 구현 시

---

## MVI 핵심 개념

```
User Action (Intent)
      ↓
   ViewModel
      ↓
  State Update
      ↓
   UI Render
```

**원칙**:
- **Single Source of Truth**: UI는 하나의 State만 관찰
- **Unidirectional Data Flow**: 데이터는 한 방향으로만 흐름
- **Immutable State**: State는 변경 불가능 (새로운 copy 생성)

---

## 패턴 구조

### 1. UI State 정의

```kotlin
data class RecipeListUiState(
    val isLoading: Boolean = false,
    val recipes: List<Recipe> = emptyList(),
    val error: String? = null,
    val filterType: FilterType = FilterType.ALL
) {
    companion object {
        val INITIAL = RecipeListUiState()
    }
}
```

**규칙**:
- `data class`로 정의 (불변성)
- 모든 UI 요소의 상태를 포함
- 기본값 제공
- `INITIAL` 상수로 초기 상태 정의

---

### 2. UI Event 정의

```kotlin
sealed interface RecipeListEvent {
    data class RecipeClicked(val recipeId: String) : RecipeListEvent
    data object LoadRecipes : RecipeListEvent
    data object RetryLoad : RecipeListEvent
    data class FilterChanged(val filterType: FilterType) : RecipeListEvent
}
```

**규칙**:
- `sealed interface` 또는 `sealed class` 사용
- 사용자 액션을 명확하게 표현
- 데이터가 필요하면 `data class`, 없으면 `data object`

---

### 3. Side Effect (Optional)

```kotlin
sealed interface RecipeListSideEffect {
    data class ShowToast(val message: String) : RecipeListSideEffect
    data class NavigateToDetail(val recipeId: String) : RecipeListSideEffect
}
```

**용도**: 일회성 이벤트 (Toast, Navigation 등)

---

### 4. ViewModel 구현

```kotlin
@HiltViewModel
class RecipeListViewModel @Inject constructor(
    private val getRecipesUseCase: GetRecipesUseCase
) : ViewModel() {

    // State
    private val _uiState = MutableStateFlow(RecipeListUiState.INITIAL)
    val uiState: StateFlow<RecipeListUiState> = _uiState.asStateFlow()

    // Side Effect (Optional)
    private val _sideEffect = Channel<RecipeListSideEffect>()
    val sideEffect = _sideEffect.receiveAsFlow()

    init {
        loadRecipes()
    }

    fun onEvent(event: RecipeListEvent) {
        when (event) {
            is RecipeListEvent.LoadRecipes -> loadRecipes()
            is RecipeListEvent.RetryLoad -> loadRecipes()
            is RecipeListEvent.RecipeClicked -> navigateToDetail(event.recipeId)
            is RecipeListEvent.FilterChanged -> updateFilter(event.filterType)
        }
    }

    private fun loadRecipes() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true, error = null) }

            getRecipesUseCase()
                .onSuccess { recipes ->
                    _uiState.update { it.copy(isLoading = false, recipes = recipes) }
                }
                .onFailure { error ->
                    _uiState.update { it.copy(isLoading = false, error = error.message) }
                }
        }
    }

    private fun navigateToDetail(recipeId: String) {
        viewModelScope.launch {
            _sideEffect.send(RecipeListSideEffect.NavigateToDetail(recipeId))
        }
    }

    private fun updateFilter(filterType: FilterType) {
        _uiState.update { it.copy(filterType = filterType) }
    }
}
```

**핵심 포인트**:
- `MutableStateFlow`는 private, `StateFlow`는 public
- `update { }`로 State 업데이트 (불변성 유지)
- Event는 `onEvent()` 함수로 통합
- Coroutine은 `viewModelScope` 사용

---

### 5. UI 연결 (Composable)

```kotlin
@Composable
fun RecipeListScreen(
    viewModel: RecipeListViewModel = hiltViewModel(),
    onRecipeClick: (String) -> Unit
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    // Side Effect 처리
    LaunchedEffect(Unit) {
        viewModel.sideEffect.collect { effect ->
            when (effect) {
                is RecipeListSideEffect.NavigateToDetail -> {
                    onRecipeClick(effect.recipeId)
                }
                is RecipeListSideEffect.ShowToast -> {
                    // Toast 표시
                }
            }
        }
    }

    RecipeListContent(
        uiState = uiState,
        onEvent = viewModel::onEvent
    )
}

@Composable
fun RecipeListContent(
    uiState: RecipeListUiState,
    onEvent: (RecipeListEvent) -> Unit
) {
    when {
        uiState.isLoading -> LoadingView()
        uiState.error != null -> ErrorView(
            message = uiState.error,
            onRetry = { onEvent(RecipeListEvent.RetryLoad) }
        )
        else -> LazyColumn {
            items(uiState.recipes) { recipe ->
                RecipeCard(
                    recipe = recipe,
                    onRecipeClick = { 
                        onEvent(RecipeListEvent.RecipeClicked(recipe.id))
                    }
                )
            }
        }
    }
}
```

**규칙**:
- `collectAsStateWithLifecycle()` 사용 (Lifecycle 인식)
- UI는 State만 관찰, Event 발행
- Side Effect는 `LaunchedEffect`에서 처리

---

## MVVM과의 통합

RecipeBeginner 프로젝트는 **MVI + MVVM** 하이브리드를 사용합니다:

- **MVVM**: ViewModel - Repository 구조
- **MVI**: UI State 단일화, Event 기반 처리

```
UI (Compose)
    ↓ Event
 ViewModel (MVI)
    ↓ UseCase 호출
Repository (MVVM)
    ↓
Data Source (Firebase, Room)
```

---

## State 업데이트 패턴

### ❌ 나쁜 예

```kotlin
// Mutable 노출
val uiState = MutableStateFlow(RecipeListUiState.INITIAL)

// 직접 변경
_uiState.value.recipes = newRecipes  // ❌ 불변성 위반
```

### ✅ 좋은 예

```kotlin
// Immutable 노출
private val _uiState = MutableStateFlow(RecipeListUiState.INITIAL)
val uiState: StateFlow<RecipeListUiState> = _uiState.asStateFlow()

// copy로 새로운 객체 생성
_uiState.update { it.copy(recipes = newRecipes) }
```

---

## 복잡한 State 업데이트

### 여러 필드 동시 업데이트

```kotlin
_uiState.update { currentState ->
    currentState.copy(
        isLoading = false,
        recipes = newRecipes,
        error = null,
        lastUpdated = System.currentTimeMillis()
    )
}
```

### 조건부 업데이트

```kotlin
private fun addRecipe(recipe: Recipe) {
    _uiState.update { currentState ->
        currentState.copy(
            recipes = currentState.recipes + recipe
        )
    }
}

private fun removeRecipe(recipeId: String) {
    _uiState.update { currentState ->
        currentState.copy(
            recipes = currentState.recipes.filter { it.id != recipeId }
        )
    }
}
```

---

## 비동기 처리 패턴

### Loading → Success/Error

```kotlin
private fun loadRecipes() {
    viewModelScope.launch {
        _uiState.update { it.copy(isLoading = true, error = null) }

        try {
            val recipes = getRecipesUseCase()
            _uiState.update { it.copy(isLoading = false, recipes = recipes) }
        } catch (e: Exception) {
            _uiState.update { it.copy(isLoading = false, error = e.message) }
        }
    }
}
```

### Result 타입 활용

```kotlin
private fun loadRecipes() {
    viewModelScope.launch {
        _uiState.update { it.copy(isLoading = true) }

        getRecipesUseCase()
            .onSuccess { recipes ->
                _uiState.update { it.copy(isLoading = false, recipes = recipes) }
            }
            .onFailure { error ->
                _uiState.update { it.copy(isLoading = false, error = error.message) }
            }
    }
}
```

---

## Testing

### ViewModel 테스트

```kotlin
@Test
fun `레시피 로딩 성공 시 UI State 업데이트`() = runTest {
    // Given
    val recipes = listOf(mockRecipe1, mockRecipe2)
    coEvery { getRecipesUseCase() } returns Result.success(recipes)

    // When
    viewModel.onEvent(RecipeListEvent.LoadRecipes)
    advanceUntilIdle()

    // Then
    val uiState = viewModel.uiState.value
    assertFalse(uiState.isLoading)
    assertEquals(recipes, uiState.recipes)
    assertNull(uiState.error)
}
```

**도구**: Turbine을 사용하면 Flow 테스트가 더 편리합니다.

```kotlin
@Test
fun `레시피 로딩 성공`() = runTest {
    viewModel.uiState.test {
        // Initial state
        assertEquals(RecipeListUiState.INITIAL, awaitItem())

        // Loading state
        viewModel.onEvent(RecipeListEvent.LoadRecipes)
        val loadingState = awaitItem()
        assertTrue(loadingState.isLoading)

        // Success state
        val successState = awaitItem()
        assertFalse(successState.isLoading)
        assertEquals(2, successState.recipes.size)
    }
}
```

---

## 참고 자료

- [Android MVI Architecture](https://developer.android.com/topic/architecture)
- [StateFlow and SharedFlow](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow)
- [Kotlin Flow](https://kotlinlang.org/docs/flow.html)
