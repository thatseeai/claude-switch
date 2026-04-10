# SessionStart hook에 Local Notification 추가하기

코딩 에이전트 구현 지시문이다.

## 문서 연혁

2026-04-10
- 최초 작성

## 목적

Claude Code CLI 및 VScode Claude Code extension에 다음 hook을 추가한다.

```
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "[ -n \"${CLAUDE_CODE_OAUTH_NAME}\" ] && { if [ \"$(uname)\" = \"Darwin\" ]; then osascript -e \"display notification \\\"${CLAUDE_CODE_OAUTH_NAME} 토큰 사용\\\" with title \\\"Claude Code\\\"\"; else notify-send \"Claude Code\" \"${CLAUDE_CODE_OAUTH_NAME} 토큰 사용\"; fi; } || true"
          }
        ]
      }
    ]
  }
}
```

## 참고 자료

- [Claude Code Docs / Hooks reference](https://code.claude.com/docs/en/hooks)

## 추가 위치

현재 작업 디렉토리 기준 .claude/settings.local.json

## 추가시 주의할 점

- 기존 다른 설정은 보존해야 한다.
- Session Start.hooks 배열에 중복 추가되지 않도록 한다.
  - command에 CLAUDE_CODE_OAUTH_NAME, osascript, notify-send의 문자열 포함 여부로 판단하면 될 듯
- Session Start.hooks 기존 hook은 유지한다.

## 추가 시점

- 기본 실행 시: os.execvp("claude", ...) 호출 전에 .claude/settings.local.json을 업데이트하면 됨
- --vscode 실행 시: .vscode/settings.json 업데이트와 함께 .claude/settings.local.json도 업데이트

## 삭제 시점

- --vscode-clear 실행 시: .vscode/settings.json 업데이트와 함께 .claude/settings.local.json도 업데이트
