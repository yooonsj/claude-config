# Claude Code Configuration

개인 Claude Code 설정 저장소. 여러 머신에서 동일한 설정을 유지하기 위해 Git으로 관리.

## 설치

### 새 머신에서 클론

```bash
# 기존 ~/.claude/ 백업
mv ~/.claude ~/.claude.bak

# 클론
git clone https://github-yooonsj/yooonsj/claude-config.git ~/.claude

# 백업에서 로컬 파일 복원 (claude.json, 세션 데이터 등)
cp ~/.claude.bak/claude.json ~/.claude/ 2>/dev/null
cp -r ~/.claude.bak/statsig ~/.claude/ 2>/dev/null
cp -r ~/.claude.bak/todos ~/.claude/ 2>/dev/null
```

### 기존 머신에서 초기 설정

```bash
cd ~/.claude
git init
git remote add origin https://github-yooonsj/yooonsj/claude-config.git
git add -A
git commit -m "init claude config"
git push -u origin main
```

## 동기화

```bash
# 변경사항 push
cd ~/.claude && git add -A && git commit -m "update" && git push

# 다른 머신에서 pull
cd ~/.claude && git pull
```

## Git 계정 설정 (멀티 계정)

### credential helper

```bash
git config --global credential.helper osxkeychain
```

### URL 치환 (계정 분리)

```bash
# 회사 계정
git config --global url."https://ivan-mercuryproject@github.com/MP-DeXP/".insteadOf "https://github-ivan/MP-DeXP/"

# 개인 계정
git config --global url."https://yooonsj@github.com/yooonsj/".insteadOf "https://github-yooonsj/yooonsj/"
```

### Keychain에 토큰 저장

Keychain Access 앱에서 각 계정별로 추가:

| 필드 | 회사 계정 | 개인 계정 |
|---|---|---|
| 키체인 항목 이름 | `https://github.com` | `https://github.com` |
| 계정 이름 | `ivan-mercuryproject` | `yooonsj` |
| 암호 | GitHub Personal Access Token | GitHub Personal Access Token |

### 사용 예시

```bash
# 회사 레포
git clone https://github-ivan/MP-DeXP/some-repo.git

# 개인 레포
git clone https://github-yooonsj/yooonsj/some-repo.git
```

---

## 파일 구조

```
~/.claude/
├── .gitignore          # 화이트리스트 (추적할 파일만 허용)
├── settings.json       # 퍼미션, 환경설정
├── README.md           # 이 문서
├── CLAUDE.md           # 글로벌 지시사항 (모든 세션에 자동 로드)
├── commands/           # 글로벌 슬래시 커맨드 (레거시)
│   └── *.md
└── skills/             # 글로벌 스킬 (권장, 자동 호출 가능)
    └── <name>/
        └── SKILL.md
```

### Git 추적 대상

| 파일 | 추적 | 설명 |
|---|---|---|
| `settings.json` | O | 퍼미션, 환경설정 (공통) |
| `CLAUDE.md` | O | 글로벌 지시사항 |
| `commands/**` | O | 슬래시 커맨드 |
| `skills/**` | O | 스킬 |
| `README.md` | O | 설명 문서 |
| `settings.local.json` | X | 머신별 로컬 설정 |
| `claude.json` | X | CLI 내부 설정, 인증 정보 |
| `statsig/` | X | 내부 데이터 |
| `todos/` | X | 내부 데이터 |

---

## settings.json 설명

공식 레퍼런스: https://code.claude.com/docs/en/settings

### permissions

퍼미션 평가 순서: **deny → ask → allow**

#### defaultMode: "bypassPermissions"

모든 퍼미션 프롬프트를 건너뜀. deny/ask 규칙은 이 모드에서도 적용.

#### deny (절대 차단)

| 규칙 | 이유 |
|---|---|
| `Bash(mkfs:*)` | 디스크/파티션 포맷 |
| `Bash(dd:*)` | 저수준 디스크 쓰기 |
| `Bash(shutdown:*)` | 시스템 종료 |
| `Bash(reboot:*)` | 시스템 재부팅 |

#### ask (확인 후 허용)

**파일 삭제**

| 규칙 | 이유 |
|---|---|
| `Bash(rm -rf:*)` | 재귀적 강제 삭제 |
| `Bash(rm -r:*)` | 재귀적 삭제 |

**권한/소유권**

| 규칙 | 이유 |
|---|---|
| `Bash(sudo:*)` | 루트 권한 실행 |
| `Bash(chmod:*)` | 파일 권한 변경 |
| `Bash(chown:*)` | 파일 소유권 변경 |

**프로세스 제어**

| 규칙 | 이유 |
|---|---|
| `Bash(kill:*)` | 프로세스 종료 |
| `Bash(killall:*)` | 이름으로 프로세스 일괄 종료 |
| `Bash(pkill:*)` | 패턴으로 프로세스 종료 |

**네트워크**

| 규칙 | 이유 |
|---|---|
| `Bash(ssh:*)` | 원격 서버 접속 |
| `Bash(scp:*)` | 원격 파일 전송 |
| `Bash(curl:*)` | HTTP 요청 |
| `Bash(wget:*)` | 파일 다운로드 |

**Git 위험 명령**

| 규칙 | 이유 |
|---|---|
| `Bash(git push:*)` | 원격 저장소에 push |
| `Bash(git reset --hard:*)` | 변경사항 완전 삭제 |
| `Bash(git clean:*)` | 추적되지 않은 파일 삭제 |
| `Bash(git force:*)` | force push 등 강제 명령 |

**컨테이너 / Kubernetes**

| 규칙 | 이유 |
|---|---|
| `Bash(docker rm:*)` | 컨테이너 삭제 |
| `Bash(docker rmi:*)` | 이미지 삭제 |
| `Bash(docker system prune:*)` | 미사용 리소스 일괄 정리 |
| `Bash(kubectl delete:*)` | K8s 리소스 삭제 |
| `Bash(helm uninstall:*)` | Helm 릴리스 제거 |

**글로벌 패키지 설치**

| 규칙 | 이유 |
|---|---|
| `Bash(npm install -g:*)` | npm 글로벌 설치 |
| `Bash(brew install:*)` | Homebrew 패키지 설치 |
| `Bash(brew uninstall:*)` | Homebrew 패키지 제거 |

**macOS 시스템**

| 규칙 | 이유 |
|---|---|
| `Bash(defaults write:*)` | macOS 시스템 설정 변경 |
| `Bash(launchctl:*)` | 데몬/에이전트 관리 |
| `Bash(networksetup:*)` | 네트워크 설정 변경 |

**민감 파일**

| 규칙 | 이유 |
|---|---|
| `Read(.env)` | 환경변수 파일 (API 키, 시크릿) |
| `Read(.env.*)` | .env.local, .env.production 등 |

### 기타 설정

| 설정 | 값 | 설명 |
|---|---|---|
| `alwaysThinkingEnabled` | `true` | Extended thinking 항상 활성화 |
| `cleanupPeriodDays` | `365` | 대화 히스토리 보존 1년 (기본 30일) |
| `enableAllProjectMcpServers` | `false` | 외부 프로젝트 MCP 자동 활성화 방지 |
| `attribution.commits` | `true` | 커밋에 Co-Authored-By 추가 |
| `attribution.pullRequests` | `true` | PR에 Claude Code 푸터 포함 |
| `env.BASH_DEFAULT_TIMEOUT_MS` | `"60000"` | Bash 타임아웃 60초 |
| `env.MCP_TIMEOUT` | `"30000"` | MCP 서버 타임아웃 30초 |
| `skipDangerousModePermissionPrompt` | `true` | bypassPermissions 모드 실행 시 경고 다이얼로그 건너뜀 |

### statusLine

```json
"statusLine": {
  "type": "command",
  "command": "node /Users/ivan/.claude/hud/omc-hud.mjs"
}
```

Oh My ClaudeCode(omc) HUD 스크립트. 터미널 하단에 컨텍스트 윈도우 사용량 등 상태 정보를 표시.

> 주의: 경로가 `/Users/ivan/`으로 하드코딩되어 있어 머신별로 다를 수 있음. 필요시 `settings.local.json`으로 이동.

---

## 플러그인

### enabledPlugins

settings.json에 선언되지만 자동 설치되지 않음. 새 머신에서는 수동 설치 필요.

| 플러그인 | 마켓플레이스 | 설명 |
|---|---|---|
| `oh-my-claudecode` | `omc` | Claude Code HUD, 상태라인 등 |
| `context7` | `claude-plugins-official` | 라이브러리 문서 실시간 조회 |
| `playwright` | `claude-plugins-official` | 브라우저 자동화/테스트 |
| `security-guidance` | `claude-plugins-official` | 보안 가이드라인 |
| `ralph-loop` | `claude-plugins-official` | 반복 작업 자동화 |
| `python-development` | `claude-code-workflows` | Python 개발 워크플로우 |
| `claude-md-management` | `claude-plugins-official` | CLAUDE.md 관리 |
| `data-engineering` | `claude-code-workflows` | 데이터 엔지니어링 워크플로우 |
| `codex` | `dexp-ai-toolkits` | OpenAI Codex 연동 (DeXP 자체) |
| `example-skills` | `anthropic-agent-skills` | Anthropic 공식 스킬 예제 |

### extraKnownMarketplaces

| 마켓플레이스 | 소스 | 비고 |
|---|---|---|
| `dexp-ai-toolkits` | `git@github.com:MP-DeXP/DeXP-AI-Toolkits.git` | DeXP 자체 플러그인, autoUpdate 활성 |
| `anthropic-agent-skills` | `github:anthropics/skills` | Anthropic 공식 스킬 |

### 새 머신에서 플러그인 설치 순서

```bash
# 1. 마켓플레이스 추가
/plugin marketplace add MP-DeXP/DeXP-AI-Toolkits
/plugin marketplace add anthropics/skills

# 2. 공식 플러그인 설치
/plugin install context7
/plugin install playwright
/plugin install security-guidance
/plugin install ralph-loop
/plugin install claude-md-management

# 3. 워크플로우 플러그인 설치
/plugin install python-development
/plugin install data-engineering

# 4. DeXP 자체 플러그인 설치
/plugin install codex

# 5. Anthropic 스킬 설치
/plugin install example-skills

# 6. Oh My ClaudeCode 설치
/plugin marketplace add anthropic-agent-skills/oh-my-claudecode
/plugin install oh-my-claudecode
```

---

## skills vs commands

| 항목 | `.claude/commands/*.md` | `.claude/skills/<name>/SKILL.md` |
|---|---|---|
| 슬래시 호출 | O | O |
| 자동 호출 | X | O (description 기반) |
| 상태 | 레거시 | 권장 |

새로 만들 때는 skills를 사용할 것.

---

## 공식 문서 링크

| 주제 | URL |
|---|---|
| Settings (전체 설정 레퍼런스) | https://code.claude.com/docs/en/settings |
| Permissions (퍼미션 규칙) | https://code.claude.com/docs/en/permissions |
| Skills (슬래시 커맨드 / 스킬) | https://code.claude.com/docs/en/slash-commands |
| Hooks (가이드) | https://code.claude.com/docs/en/hooks-guide |
| Hooks (레퍼런스) | https://code.claude.com/docs/en/hooks |
| Plugins (레퍼런스) | https://code.claude.com/docs/en/plugins-reference |
| Environment Variables | https://code.claude.com/docs/en/env-vars |
| Tools (도구 목록) | https://code.claude.com/docs/en/tools-reference |
| CLI (명령줄 레퍼런스) | https://code.claude.com/docs/en/cli-reference |
| Built-in Commands | https://code.claude.com/docs/en/built-in-commands |
| Best Practices | https://code.claude.com/docs/en/best-practices |
| Common Workflows | https://code.claude.com/docs/en/common-workflows |

---

## 유용한 환경변수

`settings.json`의 `env`에 넣거나, 셸에서 `export`로 설정 가능.

| 변수 | 기본값 | 설명 |
|---|---|---|
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | ~83.5% | 자동 컴팩션 트리거 비율. 낮추면(50~70) 일찍 컴팩션되어 정보 손실 감소, 높이면(90) 더 오래 사용 가능 |
| `MAX_THINKING_TOKENS` | 모델별 상이 | Extended thinking 토큰 예산. 0으로 설정 시 비활성화 |
| `BASH_DEFAULT_TIMEOUT_MS` | 짧음 | Bash 명령 기본 타임아웃 (ms). 긴 빌드 시 늘려야 함 |
| `BASH_MAX_TIMEOUT_MS` | - | Bash 명령 최대 타임아웃 (ms) |
| `MCP_TIMEOUT` | - | MCP 서버 연결 타임아웃 (ms) |
| `MCP_TOOL_TIMEOUT` | - | MCP 도구 실행 타임아웃 (ms) |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | ~32000 | 응답 최대 토큰. 늘리면 긴 출력 가능하지만 컨텍스트 윈도우에서 차지하는 비중 증가 |
| `MAX_MCP_OUTPUT_TOKENS` | - | MCP 응답 최대 토큰 |
| `CLAUDE_CODE_SUBAGENT_MODEL` | sonnet | 서브에이전트 모델. haiku로 바꾸면 토큰 절약 |
| `DISABLE_TELEMETRY` | 0 | 1로 설정 시 텔레메트리 비활성화 |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 0 | 1로 설정 시 자동 업데이트, 버그 리포트, 에러 리포팅, 텔레메트리 모두 비활성화 |

---

## hooks 설정 예시

hooks는 `settings.json`에 직접 넣거나, 프로젝트 `.claude/settings.json`에 넣을 수 있음.

### rm -rf 차단 (trash 사용 권장)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.command // empty' | grep -qE '^rm\\s+-rf\\b' && echo '{\"decision\": \"block\", \"reason\": \"rm -rf 대신 trash 명령어를 사용하세요.\"}' && exit 2 || exit 0"
          }
        ]
      }
    ]
  }
}
```

### main/master 직접 push 차단

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.command // empty' | grep -qE 'git\\s+push\\s+.*\\b(main|master)\\b' && echo '{\"decision\": \"block\", \"reason\": \"main/master에 직접 push할 수 없습니다. feature 브랜치를 사용하세요.\"}' && exit 2 || exit 0"
          }
        ]
      }
    ]
  }
}
```

### 파일 수정 후 자동 포맷 (Python black)

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path // empty' | grep -q '\\.py$' && black $(jq -r '.tool_input.file_path') 2>/dev/null || exit 0"
          }
        ]
      }
    ]
  }
}
```

### Hook 이벤트 종류

| 이벤트 | 시점 | 차단 가능 |
|---|---|---|
| `SessionStart` | 세션 시작 시 | X |
| `SessionEnd` | 세션 종료 시 | X |
| `PreToolUse` | 도구 실행 전 | O (exit 2) |
| `PostToolUse` | 도구 실행 후 | X (이미 실행됨) |
| `Stop` | Claude 응답 완료 시 | O (계속 진행 강제 가능) |
| `UserPromptSubmit` | 사용자 입력 전송 시 | O |
| `ConfigChange` | 설정 파일 변경 시 | O |

---

## 자주 쓰는 slash command

### 내장 명령어

| 명령어 | 설명 |
|---|---|
| `/help` | 사용 가능한 모든 명령어 목록 |
| `/compact` | 컨텍스트 윈도우 수동 압축 |
| `/clear` | 대화 히스토리 초기화 |
| `/model` | 모델 변경 (sonnet, opus, haiku) |
| `/config` | 설정 UI 열기 |
| `/permissions` | 퍼미션 규칙 확인/관리 |
| `/status` | 현재 설정 소스 및 상태 확인 |
| `/context` | 컨텍스트 윈도우 사용량 확인 |
| `/resume` | 이전 세션 재개 |
| `/hooks` | hooks 인터랙티브 설정 |
| `/plugin` | 플러그인 마켓플레이스 |
| `/simplify` | 최근 변경 파일 코드 품질 개선 (내장 스킬) |
| `/batch` | 대규모 변경 병렬 처리 (내장 스킬) |

### 키보드 단축키

| 단축키 | 설명 |
|---|---|
| `Shift+Tab` | 퍼미션 모드 전환 (Default → AcceptEdits → Plan) |
| `Option+T` / `Alt+T` | Extended thinking 토글 |
| `Ctrl+G` | Plan 모드에서 플랜을 에디터로 열기 |
| `Escape` | 실행 중 일시 정지 |
| `Ctrl+C` | 중단 |
