# Claude Code PR Review — Setup Guide

A complete guide to setting up Claude as an automated PR reviewer on GitHub.

---

## Prerequisites

- GitHub repository with admin access
- Claude.ai Pro or Max subscription
- Claude Code installed locally (`npm install -g @anthropic-ai/claude-code`)

---

## Step 1 — Install the Claude GitHub App

Go to **https://github.com/apps/claude** and click **Install**.

Choose your account or organization, select the repositories you want Claude to access, then click **Install & Authorize**.

> This app grants Claude read/write access to Contents, Issues, and Pull Requests — required for posting review comments.

---

## Step 2 — Generate an OAuth Token

Claude Code action requires an OAuth token tied to your Claude.ai subscription, not a plain API key.

Open your terminal and run:

```bash
npm install -g @anthropic-ai/claude-code
claude setup-token
```

Copy the generated token — it starts with `sk-ant-oat01-...`

---

## Step 3 — Add the Token to GitHub Secrets

Go to your repository on GitHub:

```
Settings → Secrets and variables → Actions → New repository secret
```

| Name | Value |
|------|-------|
| `CLAUDE_CODE_OAUTH_TOKEN` | `sk-ant-oat01-...` |

---

## Step 4 — Create the Workflow File

Create `.github/workflows/claude-review.yml` in your repository:

```yaml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize, ready_for_review]
  issue_comment:
    types: [created]

jobs:
  review:
    runs-on: ubuntu-latest
    if: |
      github.event_name == 'pull_request' ||
      (github.event_name == 'issue_comment' && 
       contains(github.event.comment.body, '@claude') &&
       github.event.comment.user.type != 'Bot')
    permissions:
      contents: read
      pull-requests: write
      issues: write
      id-token: write

    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0

      - uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          show_full_output: true
          claude_args: "--allowedTools Bash,Read,Write,Edit"
          prompt: |
            You are reviewing GitHub PR #${{ github.event.pull_request.number }} in repo ${{ github.repository }}.

            Run these commands to get context:
            - gh pr diff ${{ github.event.pull_request.number }}
            - gh pr view ${{ github.event.pull_request.number }}

            Read CLAUDE.md for project conventions and rules.
            Post inline comments on specific lines where rules are violated.
            Always post a summary comment using:
            gh pr comment ${{ github.event.pull_request.number }} --body "..."
            Even if no issues found, write: "✅ Code review passed. No issues found."
```

> **Important:** The workflow file must be merged into your default branch (`main`) before it will run on new PRs. If you add it via a PR, it won't trigger until after that PR is merged.

---

## Step 5 — Create CLAUDE.md

Create `CLAUDE.md` in the root of your repository. This is the "brain" Claude reads before every review — it defines your project conventions, stack, and what to look for.

Example for a Vue 3 + TypeScript project:

```markdown
# CLAUDE.md

## Project Stack
- Vue 3 (Composition API, <script setup> only — NO Options API)
- TypeScript (strict mode — no `any`, no `@ts-ignore`)
- Vite
- Tailwind CSS v4
- Element Plus (auto-imported, on-demand)
- Pinia
- Vue Router 4

## Code Review Rules

### TypeScript
- Every function must have explicit return type
- No implicit `any` — flag it as a bug
- Props must be typed with `defineProps<{...}>()`
- Emits must be typed with `defineEmits<{...}>()`

### Vue Components
- Only `<script setup lang="ts">` — reject Options API
- Composables go in `src/composables/`, named `use*.ts`
- No logic directly in template — extract to computed/methods
- Components larger than 200 lines should be flagged for splitting

### Pinia
- One store per domain (no god stores)
- No direct state mutation outside actions
- Always use `storeToRefs()` when destructuring

### What to flag
- `any` type usage
- Options API usage
- Missing error handling in async functions
- Stores with direct state mutation
- Components over 200 lines
- `console.log` left in code
```

---

## Step 6 — Add Local Commands (Optional)

These slash commands let you run Claude reviews and generate PR descriptions directly from your terminal via Claude Code.

### Code Review Command

Create `.claude/commands/code-review.md`:

```markdown
You are a senior Vue 3 + TypeScript code reviewer.

## Steps
1. Run `git diff origin/main...HEAD` to get the full diff
2. Run `git diff --name-only origin/main...HEAD` to see changed files
3. Read `CLAUDE.md` for project conventions and rules
4. Review each changed file against the rules

## What to check
- No `any` type usage
- Only `<script setup lang="ts">` — flag Options API
- No business logic in template
- Components over 200 lines — flag for splitting
- No direct state mutation outside Pinia actions
- No `console.log` left in code
- Async functions have error handling

## Output format
For each issue:
- **File**: path/to/file.vue
- **Line**: ~42
- **Severity**: 🔴 Bug / 🟡 Warning / 🔵 Suggestion
- **Issue**: short description
- **Fix**: concrete suggestion

End with a **Summary**: total issues by severity.
```

Run with:
```bash
/code-review
```

---

### PR Description Command

Create `.claude/commands/pr-description.md`:

```markdown
You are a senior developer writing a clear, professional PR description with personality.

## Steps
1. Run `gh pr view --json number,title,headRefName,baseRefName` to get PR metadata
2. Run `gh pr diff` to get the full diff
3. Run `git log origin/main...HEAD --oneline` to get commit history
4. Read `CLAUDE.md` to understand project context

## Output
Post the description directly to the PR using:
`gh pr edit --body "..."`

Use this structure:

---

## 🚀 What's this PR about?
One punchy sentence describing WHAT changed and WHY.

## ✨ Changes
- 🧩 **Components** — what changed
- 🗄️ **Store / State** — what changed
- 🛣️ **Router** — what changed
- 🎨 **Styles** — what changed
- 🔧 **Config** — what changed

## 🧪 How to test
1. Do this
2. Then this
3. Expect this ✅

## 💬 Notes
- ⚠️ Breaking changes
- 🔑 New env variables
- 📦 New dependencies
- Or: "Nothing special here, smooth sailing 🛳️"

---

## Rules
- Write in English
- Be specific — mention real file names and component names
- Sound human, not robotic
- No filler phrases
```

Run with:
```bash
/pr-description
```

---

## Project File Structure

After setup your repository should look like this:

```
your-repo/
├── .claude/
│   └── commands/
│       ├── code-review.md       ← local review command
│       └── pr-description.md    ← local PR description command
├── .github/
│   └── workflows/
│       └── claude-review.yml    ← GitHub Actions workflow
└── CLAUDE.md                    ← project conventions for Claude
```

---

## How It Works

```
PR opened
    ↓
GitHub Actions triggers claude-review.yml
    ↓
Claude Code Action authenticates via OIDC + OAuth token
    ↓
Claude reads CLAUDE.md from main branch
    ↓
Claude runs gh pr diff to get changes
    ↓
Claude reviews diff against CLAUDE.md rules
    ↓
Claude posts inline comments + summary to PR
```

---

## Trigger Behaviors

| Event | What happens |
|-------|-------------|
| PR opened | Claude automatically reviews |
| New commit pushed to PR | Claude re-reviews |
| PR moved from Draft to Ready | Claude reviews |
| `@claude` in a PR comment | Claude responds to that specific request |

---

## Troubleshooting

**No comments posted after workflow runs successfully**
- Check that `show_full_output: true` is set and look at the full Action logs
- Make sure `permission_denials_count` is 0 in the result JSON
- Verify the workflow file is merged into `main`, not just in a feature branch

**"Workflow validation failed" error**
- The workflow file in your PR differs from the one on `main`
- Merge the workflow file to `main` first, then open new PRs

**"Claude Code is not installed on this repository"**
- Install the GitHub App at https://github.com/apps/claude
- Make sure you granted it access to the correct repository

**OAuth token rejected**
- Regenerate with `claude setup-token` and update the GitHub secret
- Make sure your Claude.ai subscription is active