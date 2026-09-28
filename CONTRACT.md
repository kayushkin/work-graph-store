# work-graph-store routes

Rooted at `/`, JSON unless a route says otherwise. A repo is named by
repo-store's numeric id. Times ending `_unix_nano` are nanoseconds; `committed_at`
is git's unix seconds. An unknown query parameter on `/ref-updates` is 400.

| Method | Path | What it does |
|---|---|---|
| `GET` | `/health` | `{"status":"ok"}` |
| `GET` | `/settings` | The service's settings, read-only |
| `GET` | `/ref-updates` | `{"ref_updates":[…]}`, newest first. Query: `repo_store_id`, `session_id`, `ref` (full name, `refs/heads/main`), `since_unix_nano`, `limit` (default 200, max 5000) |
| `GET` | `/repos/{repo_store_id}/graph` | The repo's newest `max_commits` (default 300, max 5000) commits across local and remote-tracking branches, in git's topological order, each with `made_by` (the session that made it, or null), and every branch with its head, `merged_into_default`, `worktree_path` and the `sessions` that moved or checked it out. 404 when repo-store has no such repo |
| `GET` | `/activity` | For each repo a session touched in the last `hours` (default 72): the local branches sessions moved or checked out, where each stands, and the touched branches since deleted. `?format=text` is the same for an agent to read |

## A ref update

`session_id` is `LLM_BRIDGE_SESSION_ID` in the git command's environment, or
`""`. `hook` is `reference-transaction` (a ref moved) or `post-checkout` (a
worktree switched to `ref`; `HEAD` when detached). `git_subcommand` is the git
command without its arguments. `repository_path` is the main clone, whichever
worktree the command ran in; `worktree_path` is where it ran. `repo_store_id`
is 0 when repo-store has no repo at `repository_path`. Only `HEAD`,
`refs/heads/*` and `refs/remotes/*` are recorded, and only moves (old ≠ new).
