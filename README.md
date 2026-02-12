# Claude Code Telegram Notification Hook

Get real-time Telegram notifications when Claude Code performs actions in your projects.

## Quick Start

### 1. Create Telegram Bot

1. Message [@BotFather](https://t.me/botfather) on Telegram
2. Send `/newbot` and follow prompts
3. Save the bot token (looks like: `1234567890:ABCdefGHI...`)

### 2. Get Your Chat ID

**Option A:** Message [@userinfobot](https://t.me/userinfobot) → Get your ID

**Option B:** 
1. Message your new bot
2. Visit: `https://api.telegram.org/botYOUR_TOKEN/getUpdates`
3. Find `"chat":{"id":YOUR_ID}`

### 3. Configure `.env`

프로젝트 루트에 `.env` 파일을 생성합니다.

```bash
# .env
TELEGRAM_BOT_TOKEN=your_bot_token
TELEGRAM_CHAT_ID=your_chat_id
```

> 바이너리는 Claude Code가 전달하는 `cwd`(프로젝트 디렉토리)부터 **최대 2단계 상위 디렉토리**까지 `.env` 파일을 탐색합니다. 가장 먼저 발견된 `.env`를 사용합니다.

### 4. Build

```bash
# Go 1.21+ 필요
cd .claude/hooks
go build -o notification-bin notification.go

# 또는 build.sh 사용
./build.sh
```

### 5. Install

```bash
# hooks 디렉토리를 ~/.claude/hooks로 복사
cp .claude/hooks/notification-bin ~/.claude/hooks/

# settings.json을 ~/.claude/에 복사 (기존 설정과 병합 필요)
cp .claude/settings.json ~/.claude/settings.json
```

### 6. Test

```bash
# .env 파일이 있는 프로젝트 디렉토리에서 실행
echo '{"cwd":"'$(pwd)'","hook_event_name":"Notification","message":"test"}' | ~/.claude/hooks/notification-bin
```

## How It Works

Claude Code triggers hooks → Go binary reads stdin JSON → Extracts `cwd` → Finds `.env` from `cwd` (up to 2 parent dirs) → Sends formatted message to Telegram

**`.env` 탐색 순서:**
1. `{cwd}/.env`
2. `{cwd}/../.env`
3. `{cwd}/../../.env`

**Message Format:**
```
🤖 Project: my-project
⏰ 2024-01-20 15:30:45
✅ Event: Notification
📌 Need Permission: true  (권한 요청 시에만 표시)
```

## Supported Events

- `Notification` - General Claude Code notifications
- `Stop` - Operation completion
- `SubagentStop` - Sub-agent task completion

## Requirements

- Go 1.21+ (for building from source)
- Or use the pre-built binary included

## Troubleshooting

| Issue | Solution |
|-------|----------|
| No notifications | `.env` 파일에 `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` 확인 |
| `.env` not found | 프로젝트 루트 또는 상위 2단계 내에 `.env` 파일 존재 확인 |
| Token errors | 토큰 정확히 복사 (대소문자 구분) |
| Permission denied | `chmod +x ~/.claude/hooks/notification-bin` 실행 |
| Group chats | 음수 chat ID 사용 (e.g., `-1001234567890`) |
| Build fails | Go 1.21+ 설치 확인 |

## Security

- Never commit tokens to git
- Add `.env` to `.gitignore`
- Rotate tokens with BotFather's `/revoke` if compromised

## License

MIT