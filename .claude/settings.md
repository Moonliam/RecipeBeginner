# Claude Code Hooks 설정

이 파일은 `.claude/settings.json`의 내용을 설명합니다.

---

## 🎯 Hooks 개요

Hooks는 Claude Code가 특정 작업을 수행할 때 자동으로 실행되는 스크립트입니다.

---

## 🚫 PreToolUse Hooks (작업 실행 전)

### 1. master 브랜치 편집 금지

**트리거**: `Edit` 또는 `Write` 도구 사용 시

**동작**:
- 현재 브랜치를 확인
- `master` 브랜치인 경우 편집 차단
- 오류 메시지 표시: "❌ master 브랜치에서는 직접 편집할 수 없습니다. feature 브랜치를 생성하세요."

**목적**: GitHub Flow 전략 강제 - master 브랜치는 항상 안정적 상태 유지

---

## ✅ PostToolUse Hooks (작업 실행 후)

### 1. Kotlin 파일 자동 포맷팅

**트리거**: `**/*.kt` 파일을 `Edit` 또는 `Write`한 경우

**동작**:
- `ktlint`가 설치되어 있으면 자동 포맷팅 실행 (`ktlint -F`)
- 설치되지 않았으면 안내 메시지만 표시

**ktlint 설치 방법**:
```bash
# macOS/Linux
brew install ktlint

# Windows (Scoop)
scoop install ktlint

# 또는 프로젝트에 Gradle 플러그인 추가
```

### 2. 테스트 파일 수정 시 안내

**트리거**: `**/src/test/**/*Test.kt` 파일을 수정한 경우

**동작**:
- 테스트 실행 명령어 안내: "🧪 테스트 실행을 원하면 `./gradlew test`를 실행하세요."

**목적**: 테스트 작성 후 실행을 잊지 않도록 리마인더 제공

---

## 🌍 환경 변수

### ANDROID_HOME
Android SDK 경로 (시스템 환경 변수에서 가져옴)

### JAVA_HOME
JDK 경로 (시스템 환경 변수에서 가져옴)

---

## 📝 Hook 응답 형식

Hooks는 JSON 형식으로 응답을 반환합니다:

```json
{
  "block": true,               // 작업 차단 (PreToolUse만)
  "message": "에러 메시지",      // 사용자에게 표시할 메시지
  "feedback": "피드백 정보",     // 차단하지 않는 피드백
  "suppressOutput": true        // 명령 출력 숨기기
}
```

### Exit Code
- `0`: 성공
- `2`: 차단 (PreToolUse에서 작업 중단)
- 기타: 비차단 에러 (경고만 표시)

---

## 🔧 향후 추가 가능한 Hooks

### 1. Gradle 빌드 성공 검증
```json
{
  "matcher": "Edit|Write",
  "pathPattern": "**/*.gradle.kts",
  "hooks": [{
    "type": "command",
    "command": "./gradlew build --dry-run",
    "timeout": 30
  }]
}
```

### 2. 커밋 메시지 한국어 검증
```json
{
  "matcher": "Bash(git:commit)",
  "hooks": [{
    "type": "command",
    "command": "bash",
    "args": ["-c", "if echo \"$CLAUDE_TOOL_ARGS\" | grep -qE '[a-zA-Z]{10,}'; then echo '{\"block\": true, \"message\": \"커밋 메시지는 한국어로 작성하세요\"}' >&2; exit 2; fi"]
  }]
}
```

### 3. 의존성 버전 자동 업데이트 알림
```json
{
  "matcher": "Write",
  "pathPattern": "**/libs.versions.toml",
  "hooks": [{
    "type": "command",
    "command": "bash",
    "args": ["-c", "echo '{\"feedback\": \"⚠️  의존성 변경 후 Gradle Sync를 실행하세요\"}'"]
  }]
}
```

---

## 🎯 Hook 활용 팁

1. **점진적 추가**: 필요한 Hook만 활성화 (처음부터 모두 추가하면 복잡)
2. **Timeout 설정**: 긴 작업은 timeout을 충분히 설정
3. **suppressOutput 사용**: 불필요한 출력은 숨기기
4. **환경 변수 활용**: `$CLAUDE_TOOL_PATH`, `$CLAUDE_TOOL_ARGS` 등

---

이 설정은 팀 전체가 일관된 코드 품질을 유지하도록 돕습니다!
