# Skills - 안드로이드 도메인 지식

Skills는 Claude Code에게 프로젝트별 패턴과 규칙을 가르치는 문서입니다.

---

## 📚 사용 가능한 Skills

### 1. [compose-patterns](compose-patterns/SKILL.md)
Jetpack Compose UI 패턴 및 모범 사례

**언제 사용**:
- Composable 함수 작성 시
- UI 상태 관리 시
- Preview, Side Effect 처리 시

### 2. [mvi-architecture](mvi-architecture/SKILL.md)
MVI (Model-View-Intent) 아키텍처 패턴

**언제 사용**:
- ViewModel 작성 시
- UI State, Event 정의 시
- 단방향 데이터 플로우 구현 시

### 3. [hilt-di](hilt-di/SKILL.md)
Hilt 의존성 주입 패턴

**언제 사용**:
- Repository, UseCase, ViewModel 주입 시
- Module 작성 시
- Scope 관리 시

### 4. [testing-patterns](testing-patterns/SKILL.md)
단위 테스트 작성 패턴 (JUnit, MockK, Turbine)

**언제 사용**:
- ViewModel, Repository, UseCase 테스트 시
- Flow 테스트 시
- Mock 객체 생성 시

---

## 🔄 Skill 활용 흐름

```
개발 요청
    ↓
Claude가 키워드 분석
    ↓
관련 Skill 자동 적용
    ↓
패턴에 맞는 코드 생성
```

**예시**:
- "RecipeListScreen Composable 만들어줘" → `compose-patterns` 적용
- "RecipeViewModel 만들어줘" → `mvi-architecture` + `hilt-di` 적용
- "RecipeRepositoryTest 작성해줘" → `testing-patterns` 적용

---

## 📝 Skill 추가 방법

1. 새 디렉토리 생성: `.claude/skills/{skill-name}/`
2. `SKILL.md` 파일 작성
3. Frontmatter에 메타데이터 추가:

```markdown
---
name: skill-name
description: 이 스킬이 언제 사용되는지 명확하게 설명 (키워드 포함)
---

# Skill 제목

## 언제 사용
- 조건 1
- 조건 2

## 패턴

### 패턴 이름
```kotlin
// 예시 코드
```
```

---

## 🎯 향후 추가 예정 Skills

- [ ] `firebase-patterns` - Firebase Auth, Firestore 사용 패턴
- [ ] `room-database` - Room 로컬 DB 패턴
- [ ] `navigation-compose` - Compose Navigation 패턴
- [ ] `coroutine-flow` - Coroutines & Flow 심화 패턴

---

Skills를 활용하면 일관된 코드 스타일과 아키텍처를 유지할 수 있습니다!
