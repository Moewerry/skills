---
name: git-commit
description: Create a Git commit with a concise Chinese Conventional Commits message after screening changes for private information and excluding documentation other than README.md by default. Use when the user asks to commit code or current changes, such as "提交代码", "提交一下", "帮我提交", "commit code", "commit changes", or "git commit".
---

# Git Commit

## Workflow

1. Confirm the current directory is inside a Git work tree:
   ```bash
   git rev-parse --is-inside-work-tree
   ```
   If it is not a Git work tree, stop and report the problem concisely.

2. Inspect staged, unstaged, and untracked changes from the repository root:
   ```bash
   git status --short
   git diff --cached --name-status
   git diff --name-status
   git ls-files --others --exclude-standard
   ```
   If there are no staged, unstaged, or untracked changes, stop and report that there are no changes to commit.

3. Select eligible changes using the Documentation Policy and Privacy Policy below. Review candidate content before staging, including untracked files and already staged content. Do not treat pre-staged files as permission to bypass either policy. Never blindly stage the entire worktree with `git add -A` or `git add .`.

   Unstage excluded paths while retaining their local contents:
   ```bash
   git restore --staged -- <excluded-paths>
   ```
   On an unborn branch, use `git rm --cached -- <excluded-paths>` instead; never omit `--cached`. Use explicit paths and do not discard local changes.

   Stage only reviewed, eligible paths (including eligible deletions):
   ```bash
   git add -- <eligible-paths>
   ```
   If no eligible changes remain, stop and report that there is nothing eligible to commit, with a brief explanation of the exclusions.

4. Review the exact staged snapshot against both policies, then analyze its intent:
   ```bash
   git diff --cached --stat
   git diff --cached --name-status
   git diff --cached
   ```
   Use stat and name-status output first for large changes, but do not skip the privacy review of less relevant files. Review complete staged versions of added or modified files, not only changed lines; use the index version for partially staged files. Inspect both paths and content across renames. Do not dump suspected secrets into tool output, chat, or logs; use redacted findings containing paths, locations, and categories only. If any file cannot be inspected reliably, exclude it and explain the limitation. Repeat screening after any staging or content change. Stop if unresolved conflicts exist.

5. Generate exactly one commit message using the rules below.

6. Create the commit:
   ```bash
   git commit -m "<message>"
   ```

7. Report the created commit message and briefly list excluded paths and reasons without exposing private values. Mention any remaining local changes.

## Documentation Policy

- Commit normal source code, configuration, assets, and other project implementation files as appropriate. This policy excludes other documentation; it is not a rule to commit only README.md.
- Allow documentation whose basename is exactly `README.md`, case-insensitively, at any directory depth. README files must still pass the privacy review.
- By default, exclude additions, edits, deletions, and renames of other explanatory documents: Markdown/MDX, reStructuredText, text documentation, Word/PDF documents, plans, reports, notes, and documentation directories. Classify by purpose as well as extension; a code file in a documentation site can still be documentation, while a runtime text asset is not automatically a document. Treat a rename involving an excluded document as one excluded change; do not stage only one side.
- Files that define a skill or drive tools, such as `SKILL.md`, required skill references, agent instructions, and prompt templates, are functional project inputs and remain eligible when implementing those features. Do not use this exception to include unrelated notes or reports.
- Include otherwise excluded documentation only when the user explicitly requests those documents or explicitly requests including all documentation. A generic “commit everything” or an existing staged file is not a documentation exception.
- Leave excluded documents on disk and outside this commit. Do not delete them or alter tracked-file history to hide them. Do not modify `.gitignore` unless requested.

## Privacy Policy

- Screen every candidate file, including README.md, source code, configuration, fixtures, and assets. Look for real API keys, passwords, access/refresh tokens, session cookies, authorization headers, private keys, credential-bearing URLs, private environment files, personal contact/identity/payment data, internal addresses, and private local paths. Distinguish public project metadata, deliberate dummy values, and environment-variable references from actual private values using context; a keyword match alone is not proof.
- Automatically exclude files containing private information from the commit by default; no confirmation is needed to exclude them. Preserve local contents. If safe and private changes share a file, exclude the whole file rather than silently editing working credentials or deleting user data. Suspected information that cannot be resolved confidently is also excluded and reported as uncertain.
- A deletion that removes private information may be committed after review; avoid printing the removed values. A new commit does not erase existing Git history. If retained history contains private data that would be uploaded, stop before pushing and report that history needs separate remediation.
- Include private information only when the user explicitly requests the specific information or affected files be retained after the privacy category has been identified. Generic authorization to commit/push, include all files, or include documentation does not authorize including private information. Scope exceptions to what the user actually requested. Never repeat raw private values in reports or commit messages.
- Do not automatically rewrite commits, force push, rotate credentials, delete files, or replace values with invented configuration. If a check cannot complete, explain the blocker instead of claiming the changes are safe.
- Do not run the project, builds, compilers, tests, or syntax checks unless explicitly requested; provide the relevant command if needed.

## Message Rules

Use exactly this format:

```text
<type>: <description>
```

- `<type>` must be exactly one of: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, `ci`, `build`
- Include one space after the colon.
- Write `<description>` in concise Chinese.
- Do not end with punctuation.
- Do not include a scope, body, footer, bullet list, markdown, quotes, or explanations in the commit message.

## Type Selection

- `feat`: 新增功能、页面、接口、用户可见能力
- `fix`: 修复缺陷、错误行为、异常、兼容性问题
- `docs`: 仅文档、注释说明、示例文本修改
- `style`: 仅格式、空白、排版、lint 自动格式化，且不影响行为
- `refactor`: 重构实现、拆分结构、调整命名或组织方式，且无功能变化
- `perf`: 性能优化、加载速度、缓存、渲染或查询效率改善
- `test`: 新增、修改或修复测试用例、测试工具、mock
- `chore`: 辅助脚本、维护性任务、配置杂项、仓库清理
- `ci`: CI/CD 工作流、流水线、发布自动化配置
- `build`: 构建系统、打包配置、依赖版本、锁文件修改

When multiple types apply, choose the type that best describes the primary intent. Prefer user-visible intent over incidental file changes.

## Description Guidance

- Use a short verb-object Chinese phrase, such as `新增用户登录页面` or `修复验证码校验失败的问题`.
- Mention the most important changed behavior or artifact, not implementation trivia.
- Keep it specific enough to distinguish the commit from nearby changes.
- Avoid vague descriptions such as `更新代码`, `修改文件`, `调整逻辑`, or `优化项目`.
