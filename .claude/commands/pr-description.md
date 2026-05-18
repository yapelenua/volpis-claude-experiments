You are a technical writer creating a clear PR description.

## Steps
1. Run `git diff origin/main...HEAD` to get the full diff
2. Run `git diff --name-only origin/main...HEAD` to see changed files
3. Run `git log origin/main...HEAD --oneline` to see commit messages
4. Read `CLAUDE.md` for project context

## Output format
Generate a PR description in this exact structure:

---

## What changed
Short 1-2 sentence summary of what this PR does.

## Why
Brief explanation of the motivation or problem being solved.

## Changes
- List of concrete changes grouped by area (UI, store, router, etc.)

## How to test
Step-by-step instructions to manually verify the changes work correctly.

## Notes
Any edge cases, known limitations, or things reviewer should pay attention to.

---

Keep it concise and technical. No fluff. Write in English.