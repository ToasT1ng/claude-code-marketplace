---
name: git-pr-generator
description: Create a GitHub pull request using gh CLI with a structured PR template. Use this skill whenever the user wants to create or generate a PR — in English or Korean. Trigger on English phrases like "create pr", "make pr", "generate pr", "open pr", "submit pr", "make a pull request", "create a pull request", "open a pull request", "generate a pull request", "make pull request", "pr please", "pr 만들어줘", "pr 생성해줘", "pr 생성", "pr 열어줘", "pr 올려줘", "pr 만들자", "pull request 만들어줘", "pull request 생성", "풀리퀘 만들어줘", "풀리퀘 올려줘", "풀리퀘 생성", "PR 생성해줘", "PR 만들어". Always use this skill when the user says anything about creating, making, or generating a PR or pull request.
---

Create a GitHub pull request for the current branch using `gh pr create`.

## Steps

1. **Gather context** — run these commands to understand what's changing:
   ```bash
   git rev-parse --abbrev-ref HEAD          # current branch name
   BASE=$(gh repo view --json defaultBranchRef -q '.defaultBranchRef.name' 2>/dev/null || echo "main")
   git log origin/$BASE..HEAD --oneline     # commits on this branch
   git diff origin/$BASE..HEAD --stat       # files changed
   ```
   If the branch has no commits ahead of the base, tell the user and stop.

2. **Determine the base branch** — default to `main`. If the repo uses `master` or another base, detect it:
   ```bash
   gh repo view --json defaultBranchRef -q '.defaultBranchRef.name'
   ```

3. **Craft the PR title** — derive a concise title (under 70 characters) from the branch name and commit messages. Strip the branch type prefix (e.g., `feature/`, `fix/`) to get a human-readable phrase.

4. **Fill in the PR template** — use the template below, replacing the placeholders based on the actual diff and commits.

5. **Create the PR** using exactly this command shape:
   ```bash
   gh pr create \
     --title "<title>" \
     --body "$(cat <<'EOF'
   <filled-template>
   EOF
   )"
   ```
   Do not add `--draft`, `--assignee`, `--label`, or other flags unless the user explicitly asks.

6. **Report the result** — output the PR URL returned by `gh pr create`.

---

## PR Template

Always use this exact structure. Fill in each section from the diff and commit log:

```
## Summary


## Changes
- 


## Test Plan
- [ ] 
```

If there is something meaningful to note (breaking changes, follow-ups, known limitations), append a `## Notes` section after `## Test Plan`. Otherwise omit it entirely — do not include an empty `## Notes` heading.

**Language rule:**
- Default to English for all content.
- If the user explicitly requests a specific language (e.g., "한국어로 써줘", "write in Korean"), write the content of each section in that language — but always keep the section headings (`## Summary`, `## Changes`, etc.) in English.

**Filling guidance:**
- **Summary**: 2-3 sentences — what changed and why.
- **Changes**: One bullet per logical change. Reference file names where helpful. Keep each bullet under one line.
- **Test Plan**: At least one checkbox. Derive from the type of change — a bug fix gets "Reproduce the bug, verify it's gone"; a new feature gets a smoke-test step.
- **Notes**: Only include when there's something genuinely worth calling out (breaking change, known issue, follow-up ticket, deploy steps).

---

## Rules

- Always use `gh pr create` — no alternatives.
- Pass the body via a heredoc (`<<'EOF' ... EOF`), never via a temp file.
- Title must be plain text, no emoji, under 70 characters.
- Base branch is `main` unless detected otherwise.
- Do not explain the gh command output. Just show the PR URL.
