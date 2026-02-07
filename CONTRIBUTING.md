# Contributing Guide

RecipeBeginner 프로젝트에 기여하는 방법을 안내합니다.

---

## 📋 브랜치 전략: GitHub Flow

이 프로젝트는 **GitHub Flow** 브랜치 전략을 따릅니다.

### GitHub Flow 개요

GitHub Flow는 Git Flow보다 단순한 브랜치 전략으로, 지속적인 배포(CI/CD)에 적합합니다.

**핵심 원칙:**
- `master` 브랜치는 항상 배포 가능한 안정적인 상태 유지
- 모든 작업은 `master`에서 브랜치를 생성하여 진행
- Feature, hotfix 구분 없이 우선순위로 관리
- Pull Request를 통한 코드 리뷰 필수

---

## 🔄 워크플로우

### 1. 브랜치 생성

새로운 기능 개발이나 버그 수정 시, **항상 `master` 브랜치에서 새 브랜치를 생성**합니다.

```bash
# master 브랜치를 최신으로 업데이트
git checkout master
git pull origin master

# 새 브랜치 생성 (명확하고 설명적인 이름 사용)
git checkout -b feature/recipe-list-ui
git checkout -b fix/timer-crash-on-pause
git checkout -b refactor/repository-layer
```

**브랜치 네이밍 컨벤션:**
- `feature/기능명` - 새로운 기능 추가
- `fix/버그명` - 버그 수정
- `refactor/리팩토링내용` - 코드 리팩토링
- `docs/문서내용` - 문서 작업
- `test/테스트내용` - 테스트 추가/수정

**중요:** 브랜치 이름은 **무엇을 하는지 명확하게** 드러내야 합니다.

---

### 2. 개발 & 커밋 & 푸시

작업 중에는 **자주 커밋하고 원격 브랜치에 푸시**합니다.

```bash
# 작업 내용 커밋 (커밋 메시지는 상세하게)
git add .
git commit -m "Add recipe list UI with LazyColumn

- Implement RecipeListScreen composable
- Add RecipeCard component with image and title
- Integrate with RecipeViewModel
- Add loading and error states"

# 원격 브랜치에 푸시 (자주 push하여 백업 & 공유)
git push origin feature/recipe-list-ui
```

**커밋 메시지 가이드:**
- 첫 줄: 50자 이내의 간결한 요약 (명령문 형태)
- 빈 줄 추가
- 본문: 무엇을, 왜 변경했는지 상세히 설명
- 관련 이슈가 있다면 `Fixes #123` 형식으로 참조

**예시:**
```
Fix timer crash when pausing during cooking

- Add null check for timerJob before canceling
- Handle edge case when timer is already completed
- Add unit tests for pause/resume scenarios

Fixes #42
```

---

### 3. Pull Request 생성

기능 개발이 완료되었거나 피드백이 필요할 때 **Pull Request(PR)**를 생성합니다.

**PR 생성 시점:**
- ✅ 기능 완성 후 리뷰 요청
- ✅ 진행 중이지만 방향성에 대한 피드백 필요 시 (Draft PR)
- ✅ 막히는 부분이 있어 도움 요청

**PR 템플릿:**
```markdown
## 📝 변경 사항
- 레시피 목록 화면 UI 구현
- LazyColumn을 사용한 무한 스크롤
- 로딩/에러 상태 처리

## 🎯 목적
사용자가 레시피 목록을 쉽게 탐색할 수 있도록 UI 제공

## 📸 스크린샷
(화면 캡처 첨부)

## ✅ 체크리스트
- [x] 코드 스타일 준수
- [x] 단위 테스트 작성
- [x] UI 테스트 확인
- [ ] 문서 업데이트
```

---

### 4. 리뷰 & 토의

**PR이 `master`에 merge되면 곧바로 배포될 수 있으므로** 신중한 리뷰가 필요합니다.

**리뷰어 체크리스트:**
- 코드 품질 (가독성, 유지보수성)
- 아키텍처 패턴 준수 (MVI/MVVM)
- 테스트 커버리지
- 성능 이슈
- UI/UX 일관성

**리뷰 받는 사람:**
- 피드백에 열린 자세 유지
- 건설적인 토론 진행
- 필요 시 코드 수정 후 재푸시

---

### 5. 테스트

리뷰가 완료되면 **실제 환경에서 테스트**합니다.

**테스트 항목:**
- ✅ 빌드 성공 여부
- ✅ 단위 테스트 통과
- ✅ UI 테스트 통과 (필요 시)
- ✅ 실제 기기에서 동작 확인
- ✅ 성능 이슈 확인

**문제 발생 시:**
- 즉시 수정하고 재테스트
- 심각한 문제라면 merge 중단

---

### 6. 최종 Merge

모든 테스트를 통과하면 **`master` 브랜치로 merge**합니다.

```bash
# GitHub UI에서 "Squash and merge" 또는 "Merge pull request" 클릭
# 또는 CLI:
git checkout master
git merge --no-ff feature/recipe-list-ui
git push origin master
```

**Merge 후:**
- 작업 완료된 브랜치는 삭제
- 자동 배포 설정 시 즉시 배포됨 (CI/CD)

```bash
# 로컬 브랜치 삭제
git branch -d feature/recipe-list-ui

# 원격 브랜치 삭제
git push origin --delete feature/recipe-list-ui
```

---

## 🚨 주의사항

### `master` 브랜치 관리

- ❌ `master`에 직접 커밋 금지
- ❌ 리뷰 없이 merge 금지
- ✅ 항상 배포 가능한 상태 유지
- ✅ CI 테스트 통과 필수

### 긴급 버그 수정 (Hotfix)

GitHub Flow에서는 hotfix를 별도로 구분하지 않습니다.

**긴급 버그 발생 시:**
1. `master`에서 `fix/critical-bug-name` 브랜치 생성
2. 신속하게 수정
3. **우선순위를 높여** 빠르게 리뷰 & merge
4. Merge 즉시 배포

---

## 🛠️ 권장 도구

### Git Commit Template

일관된 커밋 메시지를 위해 템플릿 사용을 권장합니다.

`.gitmessage` 파일:
```
# <타입>: <제목> (50자 이내)

# 본문 (필요 시, 72자마다 줄바꿈)
# - 무엇을 변경했는지
# - 왜 변경했는지

# 관련 이슈
# Fixes #이슈번호
```

설정:
```bash
git config --local commit.template .gitmessage
```

---

## 📚 참고 자료

- [Understanding the GitHub Flow](https://guides.github.com/introduction/flow/)
- [브랜치 전략 상세 설명](https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-github-flow-git-flow-%F0%9F%93%88-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EC%A0%84%EB%9E%B5)

---

## 📞 질문/문의

작업 중 궁금한 점이나 도움이 필요하면 언제든지 PR에 코멘트를 남겨주세요!
