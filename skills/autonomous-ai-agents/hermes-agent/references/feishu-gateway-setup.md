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

## Security notes

- Do not echo App Secret in the final answer; report it as `<set>`.
- For production/group use, prefer setting `FEISHU_ALLOWED_USERS=ou_xxx,ou_yyy`.
- If allowlist is empty, anyone who can reach the bot may be able to use it depending on chat context/policy.
