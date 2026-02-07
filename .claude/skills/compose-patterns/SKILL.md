---
name: compose-patterns
description: Jetpack Compose UI 패턴 및 모범 사례. Composable 함수, 상태 관리, Side Effect, Preview 작성 시 사용.
---

# Jetpack Compose UI 패턴

## 언제 사용

- Composable 함수 작성 시
- UI 상태 호이스팅 시
- LaunchedEffect, DisposableEffect 등 Side Effect 처리 시
- @Preview 추가 시
- Material 3 컴포넌트 사용 시

---

## 핵심 원칙

### 1. State Hoisting (상태 호이스팅)

**원칙**: Composable은 stateless하게, ViewModel에서 상태 관리

**좋은 예**:
```kotlin
@Composable
fun RecipeCard(
    recipe: Recipe,
    onRecipeClick: (String) -> Unit,
    modifier: Modifier = Modifier
) {
    Card(
        onClick = { onRecipeClick(recipe.id) },
        modifier = modifier
    ) {
        // UI 구현
    }
}
```

**나쁜 예**:
```kotlin
@Composable
fun RecipeCard(recipe: Recipe) {
    var isExpanded by remember { mutableStateOf(false) } // ❌ 내부 상태
    // ...
}
```

---

### 2. Modifier 파라미터

**원칙**: `modifier`는 항상 첫 번째 파라미터 (기본값 = `Modifier`)

```kotlin
@Composable
fun RecipeImage(
    imageUrl: String,
    contentDescription: String?,
    modifier: Modifier = Modifier  // ✅ 첫 번째, 기본값
) {
    AsyncImage(
        model = imageUrl,
        contentDescription = contentDescription,
        modifier = modifier
            .fillMaxWidth()
            .aspectRatio(16f / 9f)
    )
}
```

---

### 3. Composable 네이밍

**원칙**: PascalCase, 명사형

```kotlin
// ✅ 좋은 이름
@Composable fun RecipeList()
@Composable fun TimerButton()
@Composable fun LoadingIndicator()

// ❌ 나쁜 이름
@Composable fun showRecipes()  // camelCase
@Composable fun loading()      // 소문자 시작
```

---

### 4. Preview 작성

**원칙**: 모든 Composable에 `@Preview` 추가 (Light + Dark 모드)

```kotlin
@Preview(name = "Light Mode")
@Preview(name = "Dark Mode", uiMode = Configuration.UI_MODE_NIGHT_YES)
@Composable
private fun RecipeCardPreview() {
    RecipeBeginnerTheme {
        Surface {
            RecipeCard(
                recipe = Recipe(
                    id = "1",
                    title = "김치찌개",
                    cookingTime = 30,
                    difficulty = Difficulty.EASY
                ),
                onRecipeClick = {}
            )
        }
    }
}
```

**팁**: Preview용 Mock 데이터는 별도 파일로 분리 (`PreviewData.kt`)

---

## Side Effect 패턴

### LaunchedEffect

**용도**: Coroutine 기반 비동기 작업 (API 호출, 타이머 등)

```kotlin
@Composable
fun RecipeListScreen(
    viewModel: RecipeListViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    LaunchedEffect(Unit) {
        viewModel.loadRecipes()
    }

    // UI 렌더링
}
```

**키**: `Unit`은 화면 진입 시 1회만 실행, 특정 값으로 변경하면 그 값이 바뀔 때마다 재실행

---

### DisposableEffect

**용도**: Lifecycle 이벤트 구독/해제

```kotlin
@Composable
fun TimerScreen() {
    DisposableEffect(Unit) {
        val listener = TimerListener()
        TimerService.addListener(listener)

        onDispose {
            TimerService.removeListener(listener)
        }
    }
}
```

---

## Material 3 컴포넌트

### Card

```kotlin
Card(
    onClick = { /* 클릭 핸들러 */ },
    modifier = Modifier
        .fillMaxWidth()
        .padding(16.dp),
    elevation = CardDefaults.cardElevation(defaultElevation = 2.dp)
) {
    Column(
        modifier = Modifier.padding(16.dp)
    ) {
        // 카드 내용
    }
}
```

### Button

```kotlin
Button(
    onClick = { /* 핸들러 */ },
    modifier = Modifier.fillMaxWidth()
) {
    Text("시작하기")
}
```

### TextField

```kotlin
OutlinedTextField(
    value = text,
    onValueChange = { text = it },
    label = { Text("레시피 이름") },
    modifier = Modifier.fillMaxWidth(),
    singleLine = true
)
```

---

## 리스트 패턴 (LazyColumn)

```kotlin
@Composable
fun RecipeList(
    recipes: List<Recipe>,
    onRecipeClick: (String) -> Unit,
    modifier: Modifier = Modifier
) {
    LazyColumn(
        modifier = modifier,
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        items(
            items = recipes,
            key = { it.id }  // ✅ 안정적인 key 제공
        ) { recipe ->
            RecipeCard(
                recipe = recipe,
                onRecipeClick = onRecipeClick
            )
        }
    }
}
```

**중요**: `key` 파라미터는 리스트 아이템 재정렬 시 성능 향상

---

## 로딩/에러/빈 상태 처리

```kotlin
@Composable
fun RecipeListContent(
    uiState: RecipeListUiState,
    onRecipeClick: (String) -> Unit,
    onRetry: () -> Unit
) {
    when {
        uiState.isLoading -> {
            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                CircularProgressIndicator()
            }
        }
        uiState.error != null -> {
            ErrorView(
                message = uiState.error,
                onRetry = onRetry
            )
        }
        uiState.recipes.isEmpty() -> {
            EmptyView(message = "레시피가 없습니다")
        }
        else -> {
            RecipeList(
                recipes = uiState.recipes,
                onRecipeClick = onRecipeClick
            )
        }
    }
}
```

---

## 테마 사용

```kotlin
@Composable
fun RecipeCard() {
    Card(
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surfaceVariant
        )
    ) {
        Text(
            text = "김치찌개",
            style = MaterialTheme.typography.titleMedium,  // ✅ 테마 타이포그래피
            color = MaterialTheme.colorScheme.onSurface    // ✅ 테마 색상
        )
    }
}
```

---

## Anti-Pattern (하지 말아야 할 것)

### ❌ 1. Composable 내부에서 ViewModel 생성

```kotlin
// ❌ 나쁜 예
@Composable
fun RecipeScreen() {
    val viewModel = RecipeViewModel()  // 재구성마다 새로 생성!
}

// ✅ 좋은 예
@Composable
fun RecipeScreen(
    viewModel: RecipeViewModel = hiltViewModel()
) {
    // ViewModel은 Hilt가 관리
}
```

### ❌ 2. remember 없이 mutableStateOf 사용

```kotlin
// ❌ 나쁜 예
@Composable
fun Counter() {
    var count = mutableStateOf(0)  // 재구성마다 초기화됨!
}

// ✅ 좋은 예
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
}
```

### ❌ 3. Modifier를 여러 번 전달

```kotlin
// ❌ 나쁜 예
@Composable
fun RecipeCard(
    paddingModifier: Modifier,
    sizeModifier: Modifier
) { }

// ✅ 좋은 예
@Composable
fun RecipeCard(
    modifier: Modifier = Modifier
) {
    Card(
        modifier = modifier  // 호출자가 원하는 대로 조합
    ) { }
}
```

---

## 참고 자료

- [Compose State 가이드](https://developer.android.com/jetpack/compose/state)
- [Material 3 for Compose](https://developer.android.com/jetpack/compose/designsystems/material3)
- [Side-effects in Compose](https://developer.android.com/jetpack/compose/side-effects)
