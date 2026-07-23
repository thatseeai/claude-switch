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

기본적으로 토큰은 `CLAUDE_CODE_OAUTH_TOKEN` 환경변수에 설정됩니다. `--aat` 옵션을 주면 `ANTHROPIC_AUTH_TOKEN`에 설정합니다(값은 동일).

> **주의:** 현재 Claude Code 버전은 `.claude/settings.local.json`의 `env`를 프로세스 환경변수보다 **우선**합니다. 따라서 이전에 `--vscode`로 토큰을 써둔 상태라면 일반 모드에서 다른 계정을 선택해도 무시될 수 있습니다. 이를 막기 위해 **일반 모드(`claude-switch` 단독 실행)는 실행 전 settings.local.json의 `env`에서 토큰 관련 변수 5개(`ANTHROPIC_API_KEY`, `CLAUDE_CODE_OAUTH_TOKEN`, `ANTHROPIC_AUTH_TOKEN`, `CLAUDE_CODE_OAUTH_NAME`, `ANTHROPIC_BASE_URL`)를 자동으로 제거**합니다. 즉 CLI로 계정을 전환하면 그 디렉토리의 `--vscode`용 토큰 설정은 지워지므로, VSCode에서 다시 쓰려면 `--vscode`로 재설정하세요. (`env`의 다른 키와 `hooks`는 보존됩니다.)

### 작업 디렉토리 처리

`.claude/settings.local.json`을 생성/수정하는 모든 동작(`claude-switch`, `--vscode`, `--vscode-clear`)은 실행 전 현재 디렉토리가 git 저장소인지 확인합니다.

- git 저장소이면 최상위 디렉토리로 자동 이동한 뒤 작업을 수행합니다.
- git으로 관리되지 않는 디렉토리이면 경고를 표시하고 계속 진행할지 `(y/N)` 확인을 받습니다. 엔터(기본 N)나 `n` 입력 시 중단됩니다.

일반 모드(`claude-switch`)에서는 계정 선택 후 현재 디렉토리(위 처리로 이동한 git 최상위)의 `.claude/settings.local.json`에 SessionStart 알림 hook을 추가한 뒤, 그 디렉토리에서 `claude`를 실행합니다. 따라서 하위 디렉토리에서 실행하더라도 기본적으로 git 최상위에서 `claude`가 시작됩니다.

#### git 최상위로 이동하지 않기 (`--here`)

```bash
claude-switch --here
```

`--here` 옵션을 주면 git 최상위 디렉토리로 이동하지 않고 **현재 디렉토리를 그대로 유지**한 채 작업을 수행합니다. git 저장소의 하위 디렉토리에서 그 위치 그대로 `claude`를 실행하거나, 하위 디렉토리의 `.claude/settings.local.json`을 대상으로 삼고 싶을 때 사용합니다. 일반 모드뿐 아니라 `--vscode`, `--vscode-clear`와도 함께 사용할 수 있습니다.

```bash
claude-switch --here                # 현재 하위 디렉토리에서 claude 실행
claude-switch --vscode --here       # 현재 하위 디렉토리의 settings.local.json에 토큰 설정
```

### VSCode에 토큰 설정

```bash
claude-switch --vscode
```

계정을 선택하면 현재 디렉토리의 `.claude/settings.local.json`의 `env` 객체에 토큰을 설정합니다. VSCode Claude Code 확장은 이 `env` 객체에서 환경변수를 읽습니다. 기본적으로 토큰은 `CLAUDE_CODE_OAUTH_TOKEN`에 설정되며, `--aat` 옵션을 주면 `ANTHROPIC_AUTH_TOKEN`에 설정합니다.

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "",
    "CLAUDE_CODE_OAUTH_TOKEN": "sk-ant-oat01-XXX",
    "CLAUDE_CODE_OAUTH_NAME": "personal"
  }
}
```

- 파일이 없으면 새로 생성하고, 기존 설정(`env`의 다른 키, `hooks` 등)은 유지합니다.
- git 저장소에서 `.claude/settings.local.json`이 `.gitignore`에 포함되지 않으면 `.gitignore`에 자동 추가합니다.

`--proxy [HOST:]PORT`와 함께 사용하면 `ANTHROPIC_BASE_URL`도 함께 설정합니다:

```bash
claude-switch --vscode --proxy 8080
```

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "",
    "CLAUDE_CODE_OAUTH_TOKEN": "sk-ant-oat01-XXX",
    "CLAUDE_CODE_OAUTH_NAME": "personal",
    "ANTHROPIC_BASE_URL": "http://localhost:8080"
  }
}
```

### SessionStart 알림 hook

`claude-switch` 또는 `claude-switch --vscode`로 계정을 선택하면, 현재 디렉토리의 `.claude/settings.local.json`에 SessionStart hook을 자동으로 추가합니다. Claude Code 세션이 시작될 때 OS 알림으로 어떤 계정 토큰을 사용 중인지 표시합니다.

- macOS: `osascript`를 통한 네이티브 알림
- Linux: `notify-send`를 통한 알림

기존 설정과 다른 SessionStart hook은 유지되며, 동일한 알림 hook이 이미 있으면 중복 추가하지 않습니다.

`claude-switch --vscode-clear` 실행 시 이 hook도 함께 제거됩니다.

> **참고: 중복 판단 방식**
>
> hook의 중복 여부는 command 문자열에 `CLAUDE_CODE_OAUTH_NAME`, `osascript`, `notify-send` 세 키워드가 모두 포함되어 있는지로 판단합니다(heuristic). 사용자가 직접 작성한 hook에 이 세 키워드가 모두 포함된 경우 알림 hook으로 오인되어 추가되지 않거나 `--vscode-clear` 시 의도치 않게 삭제될 수 있습니다.

### VSCode 토큰 설정 제거

```bash
claude-switch --vscode-clear
```

`.claude/settings.local.json`의 `env`에서 `ANTHROPIC_API_KEY`, `CLAUDE_CODE_OAUTH_TOKEN`, `ANTHROPIC_AUTH_TOKEN`, `CLAUDE_CODE_OAUTH_NAME`, `ANTHROPIC_BASE_URL`을 제거합니다. 사용자가 직접 추가한 다른 `env` 키는 유지하며, 제거 후 `env`가 비면 `env` 항목도 함께 삭제합니다.

또한 `.claude/settings.local.json`에서 SessionStart 알림 hook도 함께 제거합니다.

### 프록시 설정

```bash
claude-switch --proxy [HOST:]PORT
```

`ANTHROPIC_BASE_URL=http://[HOST:]PORT` 환경변수를 설정한 뒤 `claude`를 실행합니다. `HOST`를 생략하면 `localhost`를 사용합니다.

```bash
claude-switch --proxy 3001              # http://localhost:3001
claude-switch --proxy 192.168.2.12:3000 # http://192.168.2.12:3000
```

`--vscode` 옵션과 함께 사용할 수 있습니다:

```bash
claude-switch --vscode --proxy 8080
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

### 토큰 환경변수 선택 (`--aat`)

기본적으로 선택한 토큰은 `CLAUDE_CODE_OAUTH_TOKEN` 환경변수에 설정됩니다.

```bash
claude-switch --aat
```

`--aat` 옵션을 주면 토큰을 `ANTHROPIC_AUTH_TOKEN`에 설정합니다. 설정되는 값은 동일하며, 나머지 동작에는 차이가 없습니다. `--vscode`와 함께 사용하면 `.claude/settings.local.json`의 `env` 토큰 변수에도 동일하게 적용됩니다.

### 현재 디렉토리 유지 (`--here`)

```bash
claude-switch --here
```

기본적으로 `claude-switch`는 git 최상위 디렉토리로 이동한 뒤 동작합니다. `--here` 옵션을 주면 디렉토리를 이동하지 않고 현재 디렉토리를 그대로 사용합니다. 자세한 내용은 [작업 디렉토리 처리](#작업-디렉토리-처리)를 참고하세요.

### hook 프로그램 실행 (`--hook`)

```bash
claude-switch --hook ./prepare.sh
claude-switch --vscode --hook prepare.sh
```

계정을 선택한 뒤, 다음 환경변수를 설정한 상태로 지정한 `PROGRAM`을 실행합니다.

- `CS_CLAUDE_AUTH_NAME` — 선택한 계정 이름
- `CS_CLAUDE_AUTH_TOKEN` — 선택한 계정 토큰

`PROGRAM`은 `./prepare.sh`처럼 경로를 직접 지정하거나 `PATH`에 있는 실행 파일 이름으로 지정할 수 있습니다. 찾을 수 없거나 실행 권한이 없으면 에러로 종료합니다. `--here`를 쓰지 않으면 `claude-switch`는 git 최상위 디렉토리로 이동하지만, `PROGRAM`의 경로 해석과 실행 시 작업 디렉토리(cwd)는 **모두 명령을 실행한 원래 디렉토리(이동 전 위치)** 기준입니다.

- **일반 모드**: 토큰 환경변수를 설정한 뒤 `PROGRAM`을 실행하고, 성공하면 이어서 `claude`를 실행합니다.
- **`--vscode` 모드**: `.claude/settings.local.json`을 갱신한 뒤 `PROGRAM`을 실행합니다.

`PROGRAM`이 0이 아닌 종료 코드로 끝나면 `claude` 실행/이후 동작을 진행하지 않고 그 코드로 종료(abort)합니다. `--hook`은 `--vscode` 모드와 일반(claude 실행) 모드에서만 동작하며, `--list`·`--get-token`·`--usages`·`--vscode-clear`와 함께 지정하면 에러로 거부합니다.

### 도움말

```bash
claude-switch --help
```
