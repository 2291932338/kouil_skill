# OpenClaw 配置迁移快照

这个仓库现在由 OpenClaw 安全同步脚本维护，用于备份和迁移可公开/可复用的 OpenClaw 配置片段与 skills。

最近同步时间：2026-06-28T16:01:21.272936+00:00

## 已包含内容

- `skills/hermes-migrated/`：从旧 Hermes 快照迁入且未与 OpenClaw 内置技能冲突的 skills
- `root-files/`：可迁移的工作区说明/人格/本地工具说明文件（不含 MEMORY.md）
- `openclaw-config/openclaw.example.json`：脱敏后的 OpenClaw 配置示例
- `migration-reports/`：Hermes → OpenClaw 迁移报告

## 已故意排除的内容

- `MEMORY.md`、`memory/`、sessions、logs、caches、运行时状态
- `.env`、tokens、OAuth、API keys、密码、credentials、auth 文件
- GitHub/MCP/Telegram 等真实凭证

## 迁移说明

在新 OpenClaw 环境中，可复制 `skills/hermes-migrated/` 到工作区 `skills/` 下，或按需挑选单个 skill。
`openclaw.example.json` 只作结构参考，真实密钥需要在新环境中重新配置。
