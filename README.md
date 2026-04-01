# claude-switch

여러 Claude 계정 간 전환하며 Claude Code를 실행하는 CLI 도구.
선택한 계정의 API 사용량(rate limit)도 함께 표시합니다.

## 설치

```bash
# 저장소 클론
git clone https://github.com/thatseeai/claude-switch.git
cd claude-switch

# 실행 권한 부여
chmod +x claude-switch

# PATH에 추가 (선택)
ln -s "$(pwd)/claude-switch" /usr/local/bin/claude-switch
```

Python 3 표준 라이브러리만 사용하므로 별도 패키지 설치가 필요 없습니다.

## 사전 준비

### 1. Long-lived auth token 발급

각 계정에서 다음 명령을 실행하여 토큰을 발급받습니다:

```bash
claude setup-token
```

### 2. 토큰 파일 생성

`~/.claude/long-lived-auth-tokens.txt` 파일을 생성하고 이름과 토큰을 한 줄씩 작성합니다:

```
<이름> <토큰>
```

예시:

```
personal sk-ant-xxxxx
work sk-ant-yyyyy
team sk-ant-zzzzz
```

이름은 자유롭게 지정할 수 있습니다 (이메일, 별칭 등).

## 사용법

### 계정 선택 후 Claude 실행

```bash
claude-switch [ARGS]
```

↑↓ 키로 계정을 선택하면 해당 계정의 API 사용량을 표시한 뒤 `claude [ARGS]`를 실행합니다.

### VSCode에 토큰 설정

```bash
claude-switch --vscode
```

계정을 선택하면 현재 디렉토리의 `.vscode/settings.json`에 토큰을 설정합니다.

```json
{
  "claudeCode.environmentVariables": [
    { "name": "ANTHROPIC_API_KEY", "value": "" },
    { "name": "CLAUDE_CODE_OAUTH_TOKEN", "value": "sk-ant-oat01-XXX" },
    { "name": "CLAUDE_CODE_OAUTH_NAME", "value": "personal" }
  ]
}
```

- 파일이 없으면 새로 생성하고, 기존 설정은 유지합니다.
- git 저장소에서 `.vscode/settings.json`이 `.gitignore`에 포함되지 않으면 경고를 표시합니다.

`--local-proxy PORT`와 함께 사용하면 `ANTHROPIC_BASE_URL`도 함께 설정합니다:

```bash
claude-switch --vscode --local-proxy 8080
```

```json
{
  "claudeCode.environmentVariables": [
    { "name": "ANTHROPIC_API_KEY", "value": "" },
    { "name": "CLAUDE_CODE_OAUTH_TOKEN", "value": "sk-ant-oat01-XXX" },
    { "name": "CLAUDE_CODE_OAUTH_NAME", "value": "personal" },
    { "name": "ANTHROPIC_BASE_URL", "value": "http://localhost:8080" }
  ]
}
```

### VSCode 토큰 설정 제거

```bash
claude-switch --vscode-clear
```

`.vscode/settings.json`에서 `ANTHROPIC_API_KEY`, `CLAUDE_CODE_OAUTH_TOKEN`, `CLAUDE_CODE_OAUTH_NAME`, `ANTHROPIC_BASE_URL`을 제거합니다. 제거 후 환경변수가 남아있지 않으면 `claudeCode.environmentVariables` 항목도 함께 삭제합니다.

### 로컬 프록시 설정

```bash
claude-switch --local-proxy PORT
```

`ANTHROPIC_BASE_URL=http://localhost:PORT` 환경변수를 설정한 뒤 `claude`를 실행합니다. 포트는 필수 인자입니다.

`--vscode` 옵션과 함께 사용할 수 있습니다:

```bash
claude-switch --vscode --local-proxy 8080
```


### 모든 계정의 사용량 확인

```bash
claude-switch --usages
```

토큰 파일에 등록된 모든 계정의 rate limit 사용량을 표시하고 종료합니다.

표시 항목:
- Session (5h) — 5시간 윈도우 사용률
- Weekly (7d) — 7일 윈도우 사용률
- 리셋 시각 및 남은 시간
- 전체 상태 (normal / warning / rate_limited)

### 등록된 계정 목록 확인

```bash
claude-switch --list
```

토큰 파일에 등록된 계정 이름을 한 줄씩 출력합니다.

### 특정 계정의 토큰 조회

```bash
claude-switch --get-token NAME
```

해당 이름의 토큰을 stdout에 출력합니다. 셸 치환에 활용할 수 있습니다:

```bash
export CLAUDE_CODE_OAUTH_TOKEN=$(claude-switch --get-token personal)
```

### 도움말

```bash
claude-switch --help
```
