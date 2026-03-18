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

---

## skills vs commands

| 항목 | `.claude/commands/*.md` | `.claude/skills/<name>/SKILL.md` |
|---|---|---|
| 슬래시 호출 | O | O |
| 자동 호출 | X | O (description 기반) |
| 상태 | 레거시 | 권장 |

새로 만들 때는 skills를 사용할 것.
