# Hermes → OpenClaw migration report

Time: 2026-06-06T20:59:57
Source: https://github.com/2291932338/kouil_skill

## Result

- Migrated enabled workspace skills: 69
- Existing OpenClaw/bundled skills not overwritten: 13
- Hermes-specific skills archived, not enabled: 4
- Risky/red-team skills archived, not enabled: 2

## Paths

- Enabled migrated skills: `/root/.openclaw/workspace/skills/hermes-migrated`
- Archived skipped skills/config snapshot: `/root/.openclaw/workspace/_migration/hermes-archive-20260606-205957`
- OpenClaw config backup: `/root/.openclaw/workspace/_migration/backups/openclaw.json.20260606-205957.bak`

## Migrated skills

- `arxiv` from `research/arxiv`
- `research-paper-writing` from `research/research-paper-writing`
- `llm-wiki` from `research/llm-wiki`
- `polymarket` from `research/polymarket`
- `unsloth` from `mlops/training/unsloth`
- `axolotl` from `mlops/training/axolotl`
- `fine-tuning-with-trl` from `mlops/training/trl-fine-tuning`
- `dspy` from `mlops/research/dspy`
- `huggingface-hub` from `mlops/huggingface-hub`
- `weights-and-biases` from `mlops/evaluation/weights-and-biases`
- `evaluating-llms-harness` from `mlops/evaluation/lm-evaluation-harness`
- `audiocraft-audio-generation` from `mlops/models/audiocraft`
- `segment-anything-model` from `mlops/models/segment-anything`
- `outlines` from `mlops/inference/outlines`
- `serving-llms-vllm` from `mlops/inference/vllm`
- `llama-cpp` from `mlops/inference/llama-cpp`
- `youtube-content` from `media/youtube-content`
- `spotify` from `media/spotify`
- `heartmula` from `media/heartmula`
- `gif-search` from `media/gif-search`
- `webhook-subscriptions` from `devops/webhook-subscriptions`
- `claude-code` from `autonomous-ai-agents/claude-code`
- `opencode` from `autonomous-ai-agents/opencode`
- `codex` from `autonomous-ai-agents/codex`
- `yuanbao` from `yuanbao`
- `requesting-code-review` from `software-development/requesting-code-review`
- `subagent-driven-development` from `software-development/subagent-driven-development`
- `systematic-debugging` from `software-development/systematic-debugging`
- `test-driven-development` from `software-development/test-driven-development`
- `writing-plans` from `software-development/writing-plans`
- `jupyter-live-kernel` from `data-science/jupyter-live-kernel`
- `dogfood` from `dogfood`
- `findmy` from `apple/findmy`
- `imessage` from `apple/imessage`
- `github-auth` from `github/github-auth`
- `codebase-inspection` from `github/codebase-inspection`
- `github-issues` from `github/github-issues`
- `github-pr-workflow` from `github/github-pr-workflow`
- `github-repo-management` from `github/github-repo-management`
- `github-code-review` from `github/github-code-review`
- `pokemon-player` from `gaming/pokemon-player`
- `minecraft-modpack-server` from `gaming/minecraft-modpack-server`
- `native-mcp` from `mcp/native-mcp`
- `p5js` from `creative/p5js`
- `sketch` from `creative/sketch`
- `baoyu-infographic` from `creative/baoyu-infographic`
- `humanizer` from `creative/humanizer`
- `popular-web-designs` from `creative/popular-web-designs`
- `claude-design` from `creative/claude-design`
- `excalidraw` from `creative/excalidraw`
- `ideation` from `creative/creative-ideation`
- `ascii-video` from `creative/ascii-video`
- `pretext` from `creative/pretext`
- `manim-video` from `creative/manim-video`
- `touchdesigner-mcp` from `creative/touchdesigner-mcp`
- `comfyui` from `creative/comfyui`
- `baoyu-comic` from `creative/baoyu-comic`
- `songwriting-and-ai-music` from `creative/songwriting-and-ai-music`
- `architecture-diagram` from `creative/architecture-diagram`
- `design-md` from `creative/design-md`
- `ascii-art` from `creative/ascii-art`
- `pixel-art` from `creative/pixel-art`
- `linear` from `productivity/linear`
- `ocr-and-documents` from `productivity/ocr-and-documents`
- `google-workspace` from `productivity/google-workspace`
- `maps` from `productivity/maps`
- `powerpoint` from `productivity/powerpoint`
- `airtable` from `productivity/airtable`
- `web-form-automation` from `productivity/web-form-automation`

## Not overwritten because OpenClaw already has same skill name

- `blogwatcher` from `research/blogwatcher`; existing: `/usr/local/lib/nodejs/node-v23.11.0-linux-x64/lib/node_modules/openclaw/skills/blogwatcher/SKILL.md`
- `openhue` from `smart-home/openhue`; existing: `/usr/local/lib/nodejs/node-v23.11.0-linux-x64/lib/node_modules/openclaw/skills/openhue/SKILL.md`
- `himalaya` from `email/himalaya`; existing: `/usr/local/lib/nodejs/node-v23.11.0-linux-x64/lib/node_modules/openclaw/skills/himalaya/SKILL.md`
- `songsee` from `media/songsee`; existing: `/usr/local/lib/nodejs/node-v23.11.0-linux-x64/lib/node_modules/openclaw/skills/songsee/SKILL.md`
- `python-debugpy` from `software-development/python-debugpy`; existing: `/usr/local/lib/nodejs/node-v23.11.0-linux-x64/lib/node_modules/openclaw/skills/python-debugpy/SKILL.md`
- `spike` from `software-development/spike`; existing: `/usr/local/lib/nodejs/node-v23.11.0-linux-x64/lib/node_modules/openclaw/skills/spike/SKILL.md`
- `node-inspect-debugger` from `software-development/node-inspect-debugger`; existing: `/usr/local/lib/nodejs/node-v23.11.0-linux-x64/lib/node_modules/openclaw/skills/node-inspect-debugger/SKILL.md`
- `obsidian` from `note-taking/obsidian`; existing: `/usr/local/lib/nodejs/node-v23.11.0-linux-x64/lib/node_modules/openclaw/skills/obsidian/SKILL.md`
- `apple-reminders` from `apple/apple-reminders`; existing: `/usr/local/lib/nodejs/node-v23.11.0-linux-x64/lib/node_modules/openclaw/skills/apple-reminders/SKILL.md`
- `apple-notes` from `apple/apple-notes`; existing: `/usr/local/lib/nodejs/node-v23.11.0-linux-x64/lib/node_modules/openclaw/skills/apple-notes/SKILL.md`
- `xurl` from `social-media/xurl`; existing: `/usr/local/lib/nodejs/node-v23.11.0-linux-x64/lib/node_modules/openclaw/skills/xurl/SKILL.md`
- `notion` from `productivity/notion`; existing: `/usr/local/lib/nodejs/node-v23.11.0-linux-x64/lib/node_modules/openclaw/skills/notion/SKILL.md`
- `nano-pdf` from `productivity/nano-pdf`; existing: `/usr/local/lib/nodejs/node-v23.11.0-linux-x64/lib/node_modules/openclaw/skills/nano-pdf/SKILL.md`

## Archived as Hermes-specific / likely needs manual rewrite

- `hermes-agent` from `autonomous-ai-agents/hermes-agent`
- `debugging-hermes-tui-commands` from `software-development/debugging-hermes-tui-commands`
- `plan` from `software-development/plan`
- `hermes-agent-skill-authoring` from `software-development/hermes-agent-skill-authoring`

## Archived as risky/disabled

- `obliteratus` from `mlops/inference/obliteratus`
- `godmode` from `red-teaming/godmode`

## Config notes

Old Hermes config was archived for reference only and was not copied into OpenClaw. Old Hermes cron sync was not enabled automatically because it would periodically push local config externally.
