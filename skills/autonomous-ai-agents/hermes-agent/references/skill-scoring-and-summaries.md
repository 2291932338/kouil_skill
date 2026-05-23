# Hermes skill scoring, summaries, and weekly curation

This reference captures the implementation pattern for adding usage-based scoring and concise activation summaries to Hermes Agent skills.

## Goal

Reduce context pollution from over-loading skills while preserving the self-improving skill library:

- Loading a skill (`skill_view`) counts as use and increases its score.
- Each UTC day without use decays score by one point.
- A weekly curator pass writes a score report and disables low-score skills via config rather than deleting them.
- Skill index summaries prefer a short activation hint (`summary` or `metadata.hermes.summary`) over long descriptions.

## Implementation map

### `tools/skill_usage.py`

Extend the sidecar record (`~/.hermes/skills/.usage.json`) with scoring fields:

```python
"score": 0,
"score_last_decay_date": None,
"score_week_started_at": None,
"score_week_uses": 0,
"score_week_decays": 0,
"score_week_disabled_at": None,
```

Pattern:

- Keep lifecycle mutators (`set_state`, `archive_skill`, `restore_skill`, `set_pinned`, patch telemetry) protected by `is_agent_created()`.
- Allow scoring telemetry for bundled/hub skills too, so upstream skills still appear in score reports, but only agent-created skills are eligible for low-score auto-disable. Bundled/hub skills should not be added to `skills.disabled` by the scoring pass.
- On `bump_use()`:
  - `use_count += 1`
  - `score += 1`
  - `score_week_uses += 1`
  - set `last_used_at`
  - initialize `score_last_decay_date` to today so a newly used skill is not immediately decayed.
- Implement lazy decay with a function like `apply_score_decay(today=...)`, subtracting elapsed UTC days since `score_last_decay_date`.
- Implement `all_skills_score_report()` over every installed `SKILL.md`, not only agent-created skills.

Pitfall: existing tests may assert that bundled/hub skills never appear in `.usage.json`. Update that invariant: lifecycle actions still must not mutate upstream skills, but scoring telemetry may exist.

### `agent/curator.py`

The curator already runs weekly-ish via `maybe_run_curator()` from CLI/gateway tickers. Add scoring there rather than building a separate daemon.

Recommended config shape:

```yaml
curator:
  scoring:
    enabled: true
    threshold: -7
    interval_hours: 168
```

Backward-compatible flat keys can also be accepted:

```yaml
curator:
  scoring_enabled: true
  score_threshold: -7
  score_interval_hours: 168
```

Recommended behavior:

- Gate scoring reports separately from curator LLM review with `last_weekly_score_report_at`.
- Default threshold should not disable fresh zero-score skills. Use a threshold like `-7`, not `0`.
- Disable low-score agent-created skills by adding them to `skills.disabled` in `config.yaml` using `load_config()`/`save_config()`.
- Never auto-disable bundled/initial or hub-installed skills from the scoring pass; include them in score reports only. Use the `agent_created` flag from `all_skills_score_report()` as the eligibility gate.
- Never delete skill files in the scoring report path.
- Write both human and machine reports:
  - `~/.hermes/logs/skill-scores/<timestamp>/REPORT.md`
  - `~/.hermes/logs/skill-scores/<timestamp>/score-report.json`
- Include the report path in curator summary so gateway/CLI logs expose where to review it.

Pitfall: clean up orphaned `.curator_state_*.tmp` files before/around weekly report writes if tests assert atomic-write temp files are not left behind.

### `agent/skill_utils.py` and prompt builder

Prefer short activation summaries in the skills index:

1. `summary`
2. `metadata.hermes.summary`
3. `description`

Example frontmatter:

```yaml
---
name: hermes-agent
description: "Configure, extend, or contribute to Hermes Agent."
summary: "Use for Hermes CLI/config/gateway/tools/skills/provider development and troubleshooting."
metadata:
  hermes:
    summary: "Alternative nested activation hint."
---
```

Keep the summary short enough for the system prompt. In the prior implementation, truncating to ~100 chars worked better than the older 60-char description cap.

Prompt wording should instruct the model to load a skill when the compact summary clearly matches the task, not merely on keyword overlap. This reduces unnecessary skill loads and score inflation.

### `agent/prompt_builder.py`

If the skills system prompt is cached, include `.usage.json` mtime in the cache key so score/disable changes invalidate the prompt cache.

## Tests to add/update

- `tests/tools/test_skill_usage.py`
  - `bump_use` increments score and weekly use count.
  - `apply_score_decay` subtracts elapsed days.
  - `all_skills_score_report` includes unseen installed skills.
  - bundled/hub skills may get scoring telemetry but remain lifecycle-protected.
- `tests/agent/test_curator.py`
  - scoring defaults and config overrides.
  - weekly report writes `REPORT.md` and `score-report.json`.
  - low-score skills are added to `skills.disabled`.
  - report is skipped until `interval_hours` elapses unless forced.
- `tests/agent/test_prompt_builder.py`
  - `summary` wins over `description`.
  - `metadata.hermes.summary` is supported.
  - long summaries/descriptions are truncated.

Run targeted validation:

```bash
python -m compileall tools/skill_usage.py tools/skills_tool.py agent/skill_utils.py agent/prompt_builder.py agent/curator.py
python -m pytest tests/tools/test_skill_usage.py tests/agent/test_prompt_builder.py tests/agent/test_curator.py -o 'addopts=' -q
```

Expected targeted result from the original session: `200 passed`.

## Design notes

- Scoring is telemetry and config-disable based; deletion remains a user decision.
- Weekly score reporting fits Hermes' existing curator lifecycle better than adding a separate cron job.
- Default threshold `-7` means a newly installed but unused zero-score skill survives the first report and must remain unused long enough to earn a negative score.
- Prompt-summary changes and scoring reinforce each other: concise summaries help the agent avoid irrelevant loads, which improves score quality.
