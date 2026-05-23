# Hermes 配置迁移快照

这个仓库由 Hermes Agent 自动从 `/home/ubuntu/.hermes` 同步维护，用于备份和迁移当前 Hermes 的使用习惯、技能和可迁移配置。

最近同步时间：2026-05-23T16:00:57.312030+00:00

## 已包含内容

- `skills/`：当前启用/维护的 Hermes skills
- `plugins/`：用户插件（如果存在）
- `config/config.yaml`：已脱敏的 Hermes 配置
- `config/.env.example`：环境变量模板，只保留变量名，值已脱敏
- `cron/`：定时任务元数据（如果存在），明显敏感内容会被脱敏
- `root-files/`：可迁移的根目录人格、上下文和说明文件（如果存在）

## 已故意排除的内容

- `.env`、`auth.json`、OAuth tokens、API keys、密码等敏感凭证
- sessions、logs、caches、图片/音频缓存等运行时数据
- virtualenvs、`__pycache__`、生成的字节码等临时文件

## 如何用于迁移

在新机器上克隆这个仓库后，可以把需要的 `skills/`、`plugins/`、`config/config.yaml` 和 `root-files/` 内容复制到新的 Hermes 配置目录。密钥和 token 不会被同步，需要你手动根据 `config/.env.example` 重新填入新的 `.env`。

> 注意：这个仓库是迁移快照，不是完整的 Hermes 运行目录备份。这样做是为了避免泄露密钥、会话记录和运行时缓存。
