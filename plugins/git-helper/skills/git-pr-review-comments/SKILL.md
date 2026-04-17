---
name: git-pr-review-comments
description: Review the current PR and post the review as a GitHub comment. Use this skill whenever the user wants the review posted to GitHub as a PR comment. Trigger on phrases like "pr comments", "pr에 코멘트", "pr 리뷰하고 코멘트 남겨줘", "pr 리뷰하고 코멘트", "pr and comments", "post review", "pr at git", "github에 리뷰 남겨", "github 코멘트", "pr comment 남겨줘", "리뷰 코멘트 달아줘", "pr에 달아줘", "pr review comment", "leave a review on the pr", "post pr review". Use this skill (not git-pr-review-local) when the user explicitly wants the review posted to GitHub.
---

Review the current PR and post the full analysis as a GitHub PR comment.

## Steps

1. **Verify a PR exists:**
   ```bash
   gh pr view --json number,title,baseRefName,headRefName,url 2>/dev/null
   ```
   If no PR exists, tell the user and stop — do not proceed.

2. **Get the diff:**
   ```bash
   gh pr diff --color=never 2>/dev/null
   ```

3. **Analyze** the diff using the review criteria below.

4. **Write the review to a temp file** (avoids shell encoding issues):
   ```bash
   cat > /tmp/pr_review_$$.md << 'REVIEW_EOF'
   <review content here>
   REVIEW_EOF
   ```

5. **Post as a PR comment:**
   ```bash
   gh pr comment --body-file /tmp/pr_review_$$.md
   ```

6. **Clean up:**
   ```bash
   rm -f /tmp/pr_review_$$.md
   ```

7. Output the PR URL and confirm the comment was posted.

## Review criteria

Evaluate each changed file for:
- **Correctness** — logic errors, off-by-ones, wrong conditions, missing null checks
- **Security** — injection risks, exposed secrets, insecure defaults, OWASP top 10
- **Performance** — unnecessary loops, N+1 queries, blocking calls, large allocations
- **Maintainability** — naming clarity, code duplication, overly complex logic
- **Test coverage** — missing tests for new logic, untested edge cases
- **Style** — consistency with the surrounding codebase, dead code

Severity definitions:
- **Critical** — must fix before merge; correctness, security, or data-loss risk
- **Major** — should fix before merge; significant impact on quality or behavior
- **Minor** — nice to fix; style, naming, small improvements

## Comment template

Use this exact structure every time. The headers must stay identical.

```markdown
## PR Review

**Branch:** `<head>` → `<base>`  
**Files changed:** <N>  

---

### ✅ Well done
<!-- At least one genuine positive. Never skip this section. -->
- <specific thing done well>

---

### 🔴 Critical
<!-- Issues that must be fixed before merge. Omit section if none. -->
- **`<file>:<line>`** — <concise description of the problem and why it matters>

---

### 🟠 Major
<!-- Significant issues that should be fixed. Omit section if none. -->
- **`<file>:<line>`** — <description>

---

### 🟡 Minor
<!-- Small improvements. Omit section if none. -->
- **`<file>:<line>`** — <description>

---

### 📋 Summary
<2–3 sentence overall assessment. State the risk level to merge as-is.>

---
*Posted by Claude Code*
```

## Rules

- Always write the review to a file first (`/tmp/pr_review_$$.md`) and use `--body-file`. Never pass the review body directly as a shell argument — multi-line markdown with special characters will break.
- Always include at least one "Well done" item — find something real, not generic praise.
- Cite exact file and line numbers. Never say "somewhere in the code".
- Do not post if the diff is empty. Tell the user instead.
- Write review content in the same language the user used to invoke the skill (Korean → Korean review, English → English review).
- Use `$$` in the temp filename to avoid collisions with parallel runs.
