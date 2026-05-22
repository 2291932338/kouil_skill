# GitHub API fallback for network-impaired git HTTPS

Use this when `git fetch` / `git push` to `https://github.com` hangs or times out, but `https://api.github.com` is reachable and the token has write permissions.

## Detection

- `curl -I https://github.com` or git smart HTTP hangs/timeouts.
- `curl -I https://api.github.com` succeeds.
- `git push` does not fail fast with auth errors; it stalls.

Use short timeouts and `GIT_TERMINAL_PROMPT=0` so automation does not block indefinitely.

## Auth checks

Verify token identity and scopes without printing the token:

```bash
curl -sS \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/user

curl -sS \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/OWNER/REPO
```

If the API returns `403 Resource not accessible by personal access token`, do not keep retrying. For fine-grained PATs, grant the target repository and **Contents: Read and write**. For classic PATs, grant `repo`.

## Empty repo bootstrap

The Git Database `POST /git/blobs` endpoint can return `409 Git Repository is empty` for an empty repository. Bootstrap the branch first using the Contents API:

```http
PUT /repos/OWNER/REPO/contents/.hermes-sync-init
{
  "message": "chore: initialize repository",
  "content": "<base64 text>",
  "branch": "main"
}
```

Then read the ref:

```http
GET /repos/OWNER/REPO/git/ref/heads/main
```

## Efficient commit update

For many text files, avoid hundreds of individual `POST /git/blobs` calls. Build one tree with inline `content` entries:

```json
{
  "base_tree": "<base-tree-sha>",
  "tree": [
    {"path": "README.md", "mode": "100644", "type": "blob", "content": "..."},
    {"path": "config/config.yaml", "mode": "100644", "type": "blob", "content": "..."}
  ]
}
```

Then:

1. `POST /repos/OWNER/REPO/git/trees`
2. `POST /repos/OWNER/REPO/git/commits` with `parents: [<old-head-sha>]`
3. `PATCH /repos/OWNER/REPO/git/refs/heads/main` with `{"sha":"<new-commit-sha>","force":false}`

Important endpoint quirk: `GET` accepts `/git/ref/heads/main`, but updating the ref should use plural `/git/refs/heads/main`. Using singular for `PATCH` can return `404 Not Found`.

## Local repo hygiene

Keep the local worktree as a staging/export area and treat git operations as best-effort when the git endpoint is impaired:

- `git fetch origin` timeout: 30s
- `git pull --rebase origin main` timeout: 30s
- `git push -u origin main` timeout: 90s
- Fall back to the REST API when push times out.

Never print PATs, credential URLs, or Authorization headers in logs. Redact with patterns like `Authorization: [REDACTED]` and token prefixes as `[REDACTED]`.