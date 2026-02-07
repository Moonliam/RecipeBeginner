# RecipeBeginner - 요리 초보자 레시피 앱

## 📋 프로젝트 개요
요리 초보자를 위한 단계별 가이드 레시피 앱. 각 단계를 순차적으로 진행하며 타이머 기능을 통해 정확한 조리 시간을 관리할 수 있습니다.

---

## 🛠️ 기술 스택

### 언어 & 플랫폼
- **Language**: Kotlin 2.1.0
- **Platform**: Android (Min SDK 28, Target SDK 36)
- **Build Tool**: Gradle 9.1.0, AGP 8.7.3

### 아키텍처
- **UI**: Jetpack Compose
- **Pattern**: MVI + MVVM
- **DI**: Hilt 2.51.1
- **Async**: Kotlin Coroutines + Flow

### 주요 라이브러리
- **Backend**: Firebase (Auth, Firestore, Storage)
- **Local DB**: Room
- **Image Loading**: Coil 3.x
- **Navigation**: Compose Navigation
- **Testing**: JUnit, MockK, Turbine (Flow 테스트)

---

## 📁 프로젝트 구조

```
RecipeBeginner/
├── app/                    # 메인 애플리케이션 모듈
├── core/
│   ├── common/            # 공통 유틸리티, Coroutines Extensions
│   ├── ui/                # Compose 공통 컴포넌트, Theme
│   ├── domain/            # UseCase, 비즈니스 로직 모델
│   └── data/              # Repository, DataSource (Firebase, Room)
└── feature/
    ├── auth/              # 로그인/회원가입 (Firebase Auth)
    ├── recipe/            # 레시피 검색/조회 (향후 추가)
    ├── cooking/           # 단계별 요리 실행 (향후 추가)
    └── fridge/            # 냉장고 관리 (향후 추가)
```

**모듈 의존성 원칙:**
- `:app` → `:feature:*` → `:core:*`
- `feature` 모듈은 서로 독립적
- `core` 모듈은 서로 의존 가능 (data → domain → common)

---

## 🎨 코드 스타일 가이드

### Kotlin
- **JDK**: 17
- **언어 기능**: Kotlin 2.1.0 표준 (value class, context receivers 등 활용)
- **Nullable 처리**: `!!` 사용 금지, `?.` 또는 `?:` 사용
- **타입 추론**: 명확한 경우에만 타입 생략

### Compose
- **State Hoisting**: ViewModel에서 상태 관리, Composable은 stateless
- **Side Effect**: `LaunchedEffect`, `DisposableEffect` 적절히 사용
- **Preview**: 모든 Composable에 `@Preview` 추가
- **Modifier**: 파라미터 순서 - `modifier` 항상 첫 번째

### 아키텍처 패턴
- **MVI**: UI State는 Single Source of Truth (data class)
- **MVVM**: ViewModel은 UI 로직, Repository는 데이터 로직
- **UseCase**: 복잡한 비즈니스 로직은 UseCase로 분리
- **Repository**: 단일 데이터 소스 (Firebase + Room)

### 네이밍
- **파일명**: PascalCase (예: `RecipeListScreen.kt`)
- **함수명**: camelCase (예: `fun loadRecipes()`)
- **상수**: UPPER_SNAKE_CASE (예: `const val MAX_IMAGE_SIZE`)
- **Composable**: PascalCase, 명사형 (예: `RecipeCard()`)

---

## 🧪 테스트

### 단위 테스트
- **위치**: `src/test/kotlin/`
- **네이밍**: `*Test.kt` (예: `RecipeViewModelTest.kt`)
- **라이브러리**: JUnit, MockK, Turbine

**테스트 실행:**
```bash
./gradlew test
```

### UI 테스트 (추후 추가)
- **위치**: `src/androidTest/kotlin/`
- **네이밍**: `*Test.kt`
- **라이브러리**: Compose Test, Espresso

---

## 🔥 Firebase 설정

### 연동 상태
- ✅ `google-services.json` 추가 완료
- ✅ Firebase BOM, Auth, Firestore 의존성 추가
- ⏳ Authentication 구현 예정
- ⏳ Firestore 데이터 모델 설계 예정

### 주의사항
- `google-services.json`은 `.gitignore`에 포함 (절대 커밋 금지)
- Firebase 규칙은 코드와 별도로 관리

---

## 🚀 주요 명령어

### 빌드
```bash
# Clean build
./gradlew clean build

# Assemble debug APK
./gradlew assembleDebug

# Assemble release APK
./gradlew assembleRelease
```

### 실행
```bash
# Install and run on device
./gradlew installDebug
```

### 테스트
```bash
# Run all tests
./gradlew test

# Run tests for specific module
./gradlew :core:data:test
```

### 코드 품질 (추후 추가)
```bash
# Lint
./gradlew lint

# Detekt (정적 분석)
./gradlew detekt
```

---

## 📝 Git 워크플로우

### 브랜치 전략
- **GitHub Flow** 사용
- `master` 브랜치는 항상 배포 가능한 상태
- 모든 작업은 `feature/`, `fix/`, `refactor/` 브랜치에서 진행
- Pull Request 필수

### 커밋 메시지
- **언어**: 한국어
- **형식**: 제목 (50자 이내) + 본문 (상세 설명)
- **템플릿**: `.gitmessage` 참고

**예시:**
```
LazyColumn을 사용한 레시피 목록 UI 추가

- RecipeListScreen composable 구현
- RecipeCard 컴포넌트 추가
- ViewModel 연동
```

---

## 🎯 개발 중점 사항

### 현재 우선순위
1. **Material 3 디자인 시스템 구축** - 테마, 색상, 타이포그래피
2. **Firebase Auth 구현** - 구글/카카오 로그인
3. **레시피 데이터 모델 설계** - Firestore 스키마 정의
4. **냉장고 관리 기능** - CRUD 구현

### 학습 목표
- Jetpack Compose UI 심화
- 단위 테스트 작성 습관화
- Coroutine 비동기 처리 최적화
- MVI 패턴 실전 적용

---

## ⚠️ 중요 규칙

### 금지 사항
- ❌ `master` 브랜치에 직접 커밋
- ❌ 리뷰 없이 merge
- ❌ `!!` (non-null assertion) 남용
- ❌ `Any` 타입 사용 (필요 시 `unknown` 사용)
- ❌ Firebase 설정 파일 커밋
- ❌ 영어 커밋 메시지

### 필수 사항
- ✅ Pull Request를 통한 코드 리뷰
- ✅ 한국어 커밋 메시지
- ✅ 단위 테스트 작성 (ViewModel, UseCase, Repository)
- ✅ Composable에 `@Preview` 추가
- ✅ Hilt를 통한 의존성 주입

---

## 📚 참고 문서

### 공식 문서
- [Jetpack Compose](https://developer.android.com/jetpack/compose)
- [Hilt](https://dagger.dev/hilt/)
- [Kotlin Coroutines](https://kotlinlang.org/docs/coroutines-overview.html)
- [Firebase Android](https://firebase.google.com/docs/android/setup)

### 프로젝트 문서
- [CONTRIBUTING.md](CONTRIBUTING.md) - GitHub Flow 전략
- [PROJECT.md](projects/recipe-app/PROJECT.md) - 프로젝트 상세 계획 (workspace 내)
- [DEV_NOTES.md](projects/recipe-app/DEV_NOTES.md) - 개발 노트 (workspace 내)

---

## 💡 Claude Code 사용 팁

### Composable 생성 요청 시
```
RecipeCard Composable을 만들어줘. 레시피 썸네일, 제목, 난이도, 조리시간을 표시하고, 
Material 3 Card를 사용해. Preview도 추가해줘.
```

### ViewModel 생성 요청 시
```
RecipeListViewModel을 MVI 패턴으로 만들어줘. Firestore에서 레시피 목록을 가져오고,
로딩/성공/에러 상태를 관리해. Hilt 사용해.
```

### 테스트 작성 요청 시
```
RecipeRepositoryTest를 작성해줘. MockK로 Firestore를 모킹하고, 
성공/실패 케이스를 모두 테스트해.
```

---

이 파일은 Claude Code가 프로젝트를 이해하는 데 사용하는 **핵심 메모리**입니다.
프로젝트가 진행되면서 지속적으로 업데이트하세요!
