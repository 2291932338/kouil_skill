# Feishu / Lark Gateway Setup Notes

Use this reference when configuring Hermes Agent to run as a Feishu/Lark bot, especially when the user provides an App ID and App Secret directly.

## Fast path with existing credentials

1. Keep secrets in `$HERMES_HOME/.env`, not `config.yaml`:

```bash
FEISHU_APP_ID=cli_xxx
FEISHU_APP_SECRET=secret_xxx
FEISHU_DOMAIN=feishu
FEISHU_CONNECTION_MODE=websocket
```

- `FEISHU_DOMAIN=feishu` is for Feishu China.
- `FEISHU_DOMAIN=lark` is for Lark international.
- `websocket` mode is preferred because it does not require a public webhook URL.

2. Ensure the Feishu platform gets its platform toolset in `config.yaml`:

```yaml
platform_toolsets:
  feishu:
    - hermes-feishu
```

Pitfall: `hermes config set platform_toolsets.feishu '["hermes-feishu"]'` may write a YAML string, not a list, depending on CLI parsing/version:

```yaml
feishu: '["hermes-feishu"]'
```

If that happens, fix it with Python/YAML rather than trusting the string value:

```bash
python - <<'PY'
import yaml
from pathlib import Path
p = Path.home() / '.hermes' / 'config.yaml'
cfg = yaml.safe_load(p.read_text())
cfg.setdefault('platform_toolsets', {})['feishu'] = ['hermes-feishu']
p.write_text(yaml.safe_dump(cfg, sort_keys=False, allow_unicode=True))
PY
```

3. Restart and verify:

```bash
hermes gateway restart
sleep 30
hermes gateway status
tail -80 ~/.hermes/logs/gateway.log
```

Success indicators:

```text
Connecting to feishu...
[Feishu] Connected in websocket mode (feishu)
✓ feishu connected
Gateway running with 1 platform(s)
```

The Lark SDK may also print a successful `connected to wss://msg-frontier.feishu.cn/...` line.

## Dependency check

If Feishu does not connect, first verify optional dependencies in the Hermes environment:

```bash
cd ~/.hermes/hermes-agent
python - <<'PY'
try:
    import lark_oapi, qrcode, aiohttp, websockets
    print('deps_ok')
except Exception as e:
    print(type(e).__name__ + ': ' + str(e))
PY
```

## Telegram can block startup briefly but Feishu can still work

When both Telegram and Feishu credentials are present, the gateway may try Telegram first. If Telegram is blocked on the network, logs show `telegram connect timed out after 30s`; after that Hermes can still continue to Feishu and connect it. Do not declare failure just because initial `gateway status` during the timeout says stopped/activating. Wait for the gateway to finish the platform loop and check logs for Feishu success.

## Feishu app-side requirements

In the Feishu developer console:

- Enable the Bot capability.
- Subscribe to the message event `im.message.receive_v1`; a successful WebSocket connection alone only proves transport/auth, not that message events are being delivered to Hermes.
- For group chats, users must @mention the bot.
- If using interactive approval cards, subscribe to `card.action.trigger` and enable Interactive Card capability.
- For document comment replies, subscribe to `drive.notice.comment_add_v1` and grant document/drive read scopes.

## Troubleshooting: connected but no reply

First distinguish “Hermes received the message but failed to answer” from “Feishu never delivered the event”:

```bash
tail -200 ~/.hermes/logs/gateway.log | grep -E "Feishu.*Inbound|Background inbound|send failed|Dropping group|Ignoring empty"
```

- If there is no `[Feishu] Inbound ... message received` line after the user sends a test message, check Feishu developer console event subscription/install state: Bot capability enabled, `im.message.receive_v1` subscribed, app installed/published to the tenant, and bot added to the chat.
- If testing in a group, Hermes requires an @mention and the group policy must allow the sender. The Feishu adapter defaults to `FEISHU_GROUP_POLICY=allowlist`; with no `FEISHU_ALLOWED_USERS`, group messages can be dropped before they reach the agent. For initial testing, set:

```bash
FEISHU_GROUP_POLICY=open
```

  Then restart the gateway. For production, prefer `FEISHU_ALLOWED_USERS=ou_xxx,ou_yyy` or per-group rules instead of leaving groups open.
- If messages are received but processing appears flaky, temporarily disable status reactions to avoid missing reaction permissions interfering with the UX:

```bash
FEISHU_REACTIONS=false
```

Then restart and verify:

```bash
hermes gateway restart
sleep 40
hermes gateway status
tail -80 ~/.hermes/logs/gateway.log
```

Success indicators after a new user message include `[Feishu] Inbound dm message received` or `[Feishu] Inbound group message received`. If those appear but no reply is sent, continue with agent/model/send-permission debugging rather than app-side event delivery.

## Troubleshooting: approval card buttons return 200340

When the user clicks "Allow Once" / "Session" / "Always" on an approval card sent by Hermes and Feishu shows error code **200340**, the card action callback failed to reach the app. The gateway logs will show **no** `_on_card_action_trigger` or approval-related entries — the callback never arrives.

Root cause: the Feishu app lacks the **card.action.trigger** callback capability. In some Feishu console versions this event/capability is not visible or is named differently.

**Immediate workarounds (pick one):**

1. **Switch to smart approvals** — low-risk commands auto-approve, no buttons sent:
   ```bash
   hermes config set approvals.mode smart
   hermes gateway start    # see pitfall below: prefer 'start' over 'restart'
   ```
   This is the cleanest fix for Feishu users who hit 200340. The agent stops sending approval cards for routine commands entirely.

2. **Use text commands instead of buttons:**
   ```
   /approve   — approve once
   /always    — always approve
   /deny      — deny
   ```
   These go through the normal message channel and require no card callback configuration.

**Permanent fix — enable card action callbacks in the Feishu developer console:**
1. Open https://open.feishu.cn/app/<app-id>
2. Left sidebar → "事件与回调" (Events & Callbacks) → "事件配置" (Event Configuration)
3. Look for card.action.trigger (may appear as "消息卡片回调" or under "卡片交互" in some console versions)
4. If not visible, check "应用能力" (App Capabilities) → look for "消息卡片交互" or "Interactive Card" toggle
5. If neither is available (some older app types), the text command fallback is the only option
6. After enabling, publish a new app version if required by the console

**Diagnosis checklist:**
```bash
# Confirm gateway never sees the callback
tail -200 ~/.hermes/logs/gateway.log | grep -i "card_action\|approval\|approve"
# Empty = callback not reaching the app (platform-side issue)
```

## Pitfall: prefer `gateway start` over `gateway restart`

`hermes gateway restart` sends SIGUSR1 to the running process, which can cause a TEMPFAIL (exit code 75) if the process is mid-operation (e.g. waiting on an API call or approval callback). systemd then enters an auto-restart cooldown. Prefer:

```bash
hermes gateway start    # reliable: systemd starts a fresh process
```

If `restart` already caused TEMPFAIL, `start` will recover it.

## Diagnosing stuck sessions and "empty response" loops

When the user reports "飞书发消息不回复" or "模型调用工具回复为空", the root cause is often a **stuck session**, not an API format issue. A session that repeatedly times out (e.g. a PPT generation task with failing subagent delegations) will hog the conversation slot. Every new user message gets appended to the stuck session, and the agent keeps retrying the old task instead of answering the new message.

**Diagnosis path:**

```bash
# 1. Check recent sessions — look for stale ones active minutes/hours ago
hermes sessions list | head -10

# 2. Check gateway logs for response times and "empty" nudges
grep -E "response ready|empty|nudge|subagent.*timeout" ~/.hermes/logs/gateway.log | tail -20

# 3. Check the session transcript for repeated empty/nudge cycles
tail -5 ~/.hermes/sessions/<stuck-session-id>.jsonl | python3 -c "import sys,json; [print(json.dumps(json.loads(l), ensure_ascii=False)[:300]) for l in sys.stdin]"
```

**If you see repeated "You just executed tool calls but returned an empty response" nudge messages** in the session transcript, the agent is looping: model returns empty → nudge → model tries again → empty → nudge. This is a session-level problem, not an API format issue.

**Fix: delete the stuck session and restart the gateway:**

```bash
echo "y" | hermes sessions delete <stuck-session-id>
hermes gateway start    # prefer start over restart (see pitfall below)
```

**Verifying the API is healthy (not a format issue):**

The mimo-v2.5-pro model returns `content: ""` (empty string) when making tool calls — this is standard OpenAI behavior and NOT the bug. Test the full tool-call chain directly:

```bash
# Step 1: tool call request
curl -s -X POST "https://token-plan-cn.xiaomimimo.com/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $XIAOMI_API_KEY" \
  -d '{"model":"mimo-v2.5-pro","messages":[{"role":"user","content":"Search for AI news"}],"tools":[{"type":"function","function":{"name":"web_search","description":"Search web","parameters":{"type":"object","properties":{"query":{"type":"string"}},"required":["query"]}}}],"max_tokens":500}'
# Expect: finish_reason="tool_calls", content="", tool_calls=[...]

# Step 2: feed tool result back
curl -s -X POST "https://token-plan-cn.xiaomimimo.com/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $XIAOMI_API_KEY" \
  -d '{"model":"mimo-v2.5-pro","messages":[{"role":"system","content":"You are helpful."},{"role":"user","content":"Search for AI news"},{"role":"assistant","content":"","tool_calls":[{"id":"call_test","type":"function","function":{"name":"web_search","arguments":"{\"query\":\"AI news\"}"}}]},{"role":"tool","tool_call_id":"call_test","content":"{\"results\":[{\"title\":\"AI News\",\"snippet\":\"Latest AI developments\"}]}"}],"max_tokens":500}'
# Expect: finish_reason="stop", content="Here are the latest AI developments..."
```

If both steps return correct results, the API format is fine. The issue is in the session state.

## Diagnosing approval-caused delays

When Feishu replies are extremely slow (minutes to hours), check if the agent is stuck waiting for command approval:

```bash
grep -E "Inbound|response ready|approval" ~/.hermes/logs/gateway.log | tail -20
```

If you see a large gap between `Inbound ... message received` and `response ready`, the agent was blocked on approval. Switch to `approvals.mode smart` (see 200340 section above) to eliminate the bottleneck.

## Security notes

- Do not echo App Secret in the final answer; report it as `<set>`.
- For production/group use, prefer setting `FEISHU_ALLOWED_USERS=ou_xxx,ou_yyy`.
- If allowlist is empty, anyone who can reach the bot may be able to use it depending on chat context/policy.
