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
