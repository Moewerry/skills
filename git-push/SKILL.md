---
name: git-push
description: Safely push local Git commits after reconciling remote changes and screening outgoing history for private information and unrequested documentation other than README.md. Use when the user asks to push code, upload code, sync commits, or says "推送代码", "上传代码", "提交到仓库", "同步到远端", "push code", or "git push". Stop when history contains excluded content or remote integration is ambiguous.
---

# Git Push

## Workflow

1. Confirm the current directory is inside a Git work tree:
   ```bash
   git rev-parse --is-inside-work-tree
   ```
   If it is not a Git work tree, stop and report the problem concisely.

2. Confirm there is no unfinished Git operation:
   ```bash
   git status --short
   ```
   If a rebase, merge, cherry-pick, or revert is already in progress, inspect `git status` and continue only when the next required Git action is unambiguous. Otherwise stop and report the blocking state.

3. Require a clean worktree before rebasing and pushing:
   ```bash
   git status --short
   ```
   If unstaged, staged, or untracked changes exist, stop and tell the user to commit or stash them first. Do not auto-commit, auto-stash, or discard worktree changes.

4. Resolve the current branch and upstream:
   ```bash
   git branch --show-current
   git rev-parse --abbrev-ref --symbolic-full-name @{u}
   ```
   If there is no upstream branch, stop and ask the user to set an upstream or specify the remote branch. Do not guess a remote branch.

5. Fetch every remote state:
   ```bash
   git fetch --all --prune
   ```

6. Discover relevant remote branches for the current branch:
   ```bash
   branch="$(git branch --show-current)"
   git for-each-ref --format='%(refname:short)' "refs/remotes/*/$branch"
   ```
   Treat every remote branch with the same branch name as a push target, excluding symbolic references such as `origin/HEAD`. Always include the configured upstream branch. If the user explicitly names remotes or remote branches, use only those targets after verifying they exist. If no relevant remote branch exists beyond an upstream that also cannot be resolved, stop and ask the user to specify the target.

7. Compare local branch and every target remote branch:
   ```bash
   git rev-list --left-right --count <remote>/<branch>...HEAD
   ```
   Interpret output as `<behind> <ahead>` for each remote target.

8. Decide the next Git action:
   - If every target has `<behind>` equal to `0`, do not pull. Push local commits directly to each target whose `<ahead>` is greater than `0`.
   - If the upstream target has `<behind>` greater than `0`, rebase local commits onto upstream:
     ```bash
     git pull --rebase
     ```
     After the rebase, fetch again if needed and recompute ahead/behind for every target before pushing.
   - If a non-upstream target has `<behind>` greater than `0` after the upstream rebase check, stop and report that the remote contains commits not present locally. Do not merge or rebase across multiple remote histories unless the user explicitly chooses that remote integration strategy.
   - If every target has `<ahead>` equal to `0` after any required rebase, report that there are no local commits to push.

9. Handle rebase conflicts by intent:
   - Inspect conflict files with:
     ```bash
     git status --short
     git diff --name-only --diff-filter=U
     ```
   - Read the conflict hunks and surrounding code to understand both sides' intent.
   - If the correct resolution is clear from code context and preserves both compatible intents, resolve it directly.
   - If the conflict is generated or mechanical and the repository has an obvious normal regeneration command, tell the user the command to run instead of running project build, compile, test, or syntax-check commands unless explicitly allowed.
   - If multiple valid resolutions exist, stop and ask the user to choose. Present 2-4 concrete options with short tradeoffs, such as keeping local behavior, keeping remote behavior, combining both paths, or manually editing a named file.
   - Do not choose one side blindly, delete code to make conflicts disappear, or invent business logic that is not supported by surrounding code.
   - After the user chooses or after a clear direct resolution, stage resolved files and continue:
     ```bash
     git add <resolved-files>
     git rebase --continue
     ```
   - Do not run `git rebase --abort` unless the user asks.

10. Before pushing, review every outgoing commit for each target remote, including intermediate snapshots rather than only the final branch diff:
    ```bash
    git log --format=oneline <remote>/<branch>..HEAD
    git diff --name-status <remote>/<branch>...HEAD
    ```
    Inspect the file changes and added/modified file snapshots of every outgoing commit. For merge commits, also inspect merge resolutions and changes against their parents. A secret added in one commit and deleted in a later commit is still uploaded in history. Review commit messages and filenames for private information as well. Perform this check after any rebase or other history change and before the first push; check all targets before uploading to any of them.

    Apply these defaults even if the commits were created outside this skill:
    - Exclude private information unless the user explicitly requests retaining the specific information or affected files after its category is identified. Look for real credentials, tokens, private keys, cookies, personal data, internal addresses, and private paths; distinguish public metadata, placeholders, and environment-variable references using context. Generic permission to push or include all files is not an exception.
    - Exclude explanatory documentation other than files with the case-insensitive basename `README.md`, unless the user explicitly requests the documents. Apply this to additions, edits, renames, and deletions. Source code and functional inputs such as `SKILL.md`, required skill references, agent instructions, and prompt templates remain eligible. README.md and functional files still require privacy review.
    - Safe deletions of private information are eligible, but do not erase private values from earlier outgoing commits. Do not print sensitive values in tool output, chat, or logs; report only redacted categories and locations.

    If outgoing history contains excluded content, automatically withhold the affected push and report the affected commits, paths, and categories. Do not silently rewrite existing commits, create cleanup commits that leave secrets in outgoing history, or force push. Explain that the local commits need to be recreated without the excluded content, or that a specific user exception is required. If review cannot complete reliably, stop and explain the limitation. Never claim that a final clean diff proves the history is safe.

11. Push local commits to every safe target remote:
   ```bash
   git push <remote> HEAD:<branch>
   ```
   Push targets one at a time. If a push target already has `<ahead>` equal to `0`, skip it and report it as already up to date. If a push is rejected because that remote advanced again, repeat from the fetch/compare step once for all targets. If it fails again, stop and report which remote failed and why.

12. Report the final result concisely, including whether a rebase happened, which remotes were pushed, which were already up to date, and whether any remote was skipped or blocked.

## Safety Rules

- Never run destructive commands such as `git reset --hard`, `git checkout -- <file>`, `git clean`, or `git rebase --abort` unless explicitly requested.
- Never force push unless the user explicitly asks and confirms the exact branch.
- Never create commits in this workflow. If local changes are uncommitted, stop and ask the user to commit or stash them first.
- Never assume remote histories are interchangeable. If different remotes contain different unseen commits, stop and ask the user how to reconcile them.
- Prefer correctness over completion. When conflict resolution is not objectively clear, explain the viable choices and wait for the user's decision.
- Do not run project builds, compilers, tests, or syntax checks unless the user explicitly requests them. If validation would normally be useful, tell the user the command to run.

## Trigger Examples

- `推送代码`
- `上传代码到仓库`
- `把本地提交同步到远端`
- `帮我 push 一下`
- `git push`
- `push current commits`
