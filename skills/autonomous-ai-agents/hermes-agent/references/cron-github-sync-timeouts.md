# Cron GitHub Sync Timeout Workaround

Use this when a Hermes cron job runs a script that syncs config/skills to GitHub and fails with a pre-run timeout, especially around 120 seconds.

## Symptom

Cron output shows a script error like:

```text
Script timed out after 120s: /home/ubuntu/.hermes/scripts/sync_hermes_to_github.py
```

The job may still show `last_status: ok` because the agent successfully reported the failure, but the pre-run data-collection script did not complete.

## Root cause observed

In this environment, GitHub's git HTTPS/smart-HTTP endpoint can hang during `git fetch`, `git pull`, or `git push`. A script that tries those operations before falling back to the GitHub REST API can exceed Hermes cron's pre-run timeout.

This is distinct from token failure. Verify token/API access separately; a valid token can still be paired with hanging git HTTPS operations.

## Preferred fix

For cron-safe GitHub sync scripts:

1. Avoid network git operations in the cron path:
   - no `git clone`
   - no `git fetch`
   - no `git pull`
   - no `git push`
2. Prepare the local export directory with local-only git commands if useful:
   - `git init`
   - `git checkout -B main`
   - `git add/commit` for local inspectability
3. Publish using GitHub REST API endpoints instead:
   - `GET /repos/{owner}/{repo}/git/ref/heads/{branch}`
   - `POST /repos/{owner}/{repo}/git/trees`
   - `POST /repos/{owner}/{repo}/git/commits`
   - `PATCH /repos/{owner}/{repo}/git/refs/heads/{branch}`
4. Verify the full script completes under the cron pre-run budget, e.g. under 120 seconds.

## API pitfalls

- Read ref endpoint can use `/git/ref/heads/main`.
- Patch ref endpoint must use `/git/refs/heads/main`.
- Empty repositories may reject direct blob/tree creation. Bootstrap first with the Contents API:

```text
PUT /repos/{owner}/{repo}/contents/.hermes-sync-init
```

- If `POST /git/blobs` is slow or problematic, build a tree with inline `content` entries and send one `POST /git/trees` request.
- Do not print or store GitHub tokens in logs, session summaries, README files, or memory.

## Verification

Run the script with a hard timeout at or below the cron limit:

```bash
start=$(date +%s)
python /home/ubuntu/.hermes/scripts/sync_hermes_to_github.py
end=$(date +%s)
echo "duration_seconds=$((end-start))"
```

Then check the cron job:

```bash
hermes cron list
```

Expected properties:

- `enabled: true`
- `script` is a filename under `~/.hermes/scripts/`, not an absolute path
- `deliver: origin` if the user expects Feishu/Lark delivery
- recent/manual run completes successfully
