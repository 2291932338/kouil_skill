# Telegram Gateway Setup Notes

Session-derived notes for configuring Hermes Agent with Telegram.

## Minimal configuration

Telegram gateway can be enabled entirely from `~/.hermes/.env`:

```env
TELEGRAM_BOT_TOKEN=<botfather-token>
TELEGRAM_ALLOWED_USERS=<telegram-user-id>[,<another-id>]
TELEGRAM_HOME_CHANNEL=<telegram-user-id-or-chat-id>
TELEGRAM_HOME_CHANNEL_NAME=<display name>
```

`TELEGRAM_BOT_TOKEN` activates the Telegram platform in gateway config via env override. `TELEGRAM_ALLOWED_USERS` prevents open access.

Restart after changes:

```bash
hermes gateway restart
hermes gateway status
```

If `restart` leaves systemd in `activating (auto-restart)` with gateway exit status 75, wait briefly or run:

```bash
hermes gateway start
```

## Verification commands

Mask bot tokens when showing logs:

```bash
hermes gateway status
journalctl --user -u hermes-gateway.service -n 80 --no-pager | sed -E 's/(bot?[0-9]{8,}:)[-A-Za-z0-9_]+/\1***/g'
tail -120 ~/.hermes/logs/gateway.log | sed -E 's/(bot?[0-9]{8,}:)[-A-Za-z0-9_]+/\1***/g'
```

Network checks:

```bash
curl -4 -I --connect-timeout 10 https://api.telegram.org
getent hosts api.telegram.org
```

## Common pitfall: server cannot reach Telegram

On some networks (notably mainland China / restricted hosts), Hermes may be configured correctly but Telegram connection fails. Logs can show:

```text
Connecting to telegram...
Primary api.telegram.org connection failed
Fallback IP ... failed
Connect attempt 1/8 failed: Timed out
```

This indicates a network reachability/proxy issue, not necessarily a Hermes config error. Confirm with `curl https://api.telegram.org`. If Baidu or other local sites work while Telegram/Google time out, configure a proxy.

Hermes Telegram proxy env var:

```env
TELEGRAM_PROXY=socks5://127.0.0.1:7890
# or
TELEGRAM_PROXY=http://127.0.0.1:7890
```

Then restart gateway and recheck logs.

## Source-code behavior observed

`gateway/config.py` maps `telegram.allow_from` in config YAML to `TELEGRAM_ALLOWED_USERS`, but `.env` is the simplest and highest-confidence route. `TELEGRAM_BOT_TOKEN` env override creates/enables `Platform.TELEGRAM` and sets the token.
