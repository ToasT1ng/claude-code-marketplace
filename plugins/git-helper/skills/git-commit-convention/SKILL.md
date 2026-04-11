---
name: git-commit-convention
description: Write a one-line gitmoji commit message for staged changes. Use this skill whenever the user wants to commit or needs a commit message — in English or Korean. Trigger on English phrases like "commit this", "write a commit", "make a commit", "commit my changes", "commit these changes", "what should I commit", "suggest a commit message", "generate a commit message", "help me commit", "what's a good commit message", "git commit message", "commit convention", "how should I commit", "stage and commit", "write me a commit", "commit message for this", "what commit message should I use", "draft a commit", "create a commit message", "give me a commit message", "commit format", "git commit format", "I want to commit", "ready to commit", "committing this", "commit these files", "commit message please", "what do I write for my commit", "message for commit", "write commit for me". Also trigger on Korean phrases like "커밋 메시지 써줘", "커밋 메시지 만들어줘", "커밋 메시지 추천해줘", "커밋 메시지 생성해줘", "커밋 해줘", "커밋 좀 해줘", "이거 커밋해줘", "커밋 작성", "커밋 뭐라고 써", "커밋 뭐로 할까", "어떻게 커밋할까", "변경사항 커밋", "깃 커밋", "깃 커밋 메시지", "커밋 남겨줘", "커밋 컨벤션", "커밋 규칙", "커밋 형식", "커밋 도와줘", "커밋 메시지 뭐가 좋아", "커밋 메시지 어떻게 써", "커밋 메시지 추천", "커밋 뭐로 남겨", "커밋 메시지 좀", "커밋 어떻게 해", "좋은 커밋 메시지", "커밋 메시지 알려줘", "커밋 할게", "커밋하려고", "커밋 내용 써줘", "이 변경사항 커밋", "커밋 메시지 뭘로 해", "깃 커밋 메시지 써줘", "커밋 예시", "커밋 스타일", "커밋 포맷".
---

Write a gitmoji commit message for the staged changes. Commit messages are always in English.

## Steps

1. Run `git diff --staged` to see what is staged.
   - If nothing is staged, run `git diff` to check for unstaged changes.
   - If nothing is staged or unstaged, tell the user there is nothing to commit and stop.
2. Check whether the staged changes are semantically cohesive — do they belong in one commit?
   - If the changes span **unrelated concerns**, split them. See the splitting guide below.
   - If the changes are cohesive (e.g. a feature and its direct test consequences), write one message.
3. Pick one gitmoji that best represents the primary intent.
4. Write a concise one-line description in English.
5. Output only the commit message(s), nothing else.

## Format

```
<gitmoji> short description of the change
```

Use the actual unicode emoji character (e.g. `✨`), not the colon code (e.g. ~~`:sparkles:`~~).

Keep the full line under 72 characters.

## Splitting commits

Commits should be small but meaningful. The right unit is a **coherent change** — everything that belongs together conceptually. A few examples:

| What changed | How to split |
|---|---|
| New feature + its tests | One commit — tests are a direct consequence |
| Domain model change + DB migration | One commit — tightly coupled |
| Domain change + unrelated refactor in another module | Two commits |
| Business logic change + test for that logic + unrelated test fix | Two commits — group business logic with its tests, separate the unrelated fix |
| New API endpoint + documentation update | One commit |

When you decide to split, output multiple commit messages, one per line, in the order they should be committed:

```
✨ add User domain model
🔧 add users table migration
✅ add unit tests for User domain
```

The user will then stage and commit each group separately.

## Examples

**Example 1 — new feature with tests:**
Diff shows: new `OrderService.kt`, new `OrderServiceTest.kt`
Output: `✨ add order service with validation logic`

**Example 2 — bug fix:**
Diff shows: off-by-one fix in `PaginationHelper.kt`
Output: `🐛 fix off-by-one error in pagination offset`

**Example 3 — mixed unrelated changes:**
Diff shows: new `PaymentService.kt`, unrelated fix in `UserRepository.kt`
Output:
```
✨ add payment service
🐛 fix user repository query
```

## Gitmoji reference

| Emoji | Use when |
|-------|----------|
| ✨ | New feature |
| 🐛 | Bug fix |
| 🔥 | Remove code or files |
| ♻️ | Refactor |
| 📝 | Documentation |
| 🎨 | Improve structure or format |
| ✅ | Add or update tests |
| 🚀 | Performance improvement |
| 🔧 | Configuration or migration |
| 📦 | Dependency change |
| ⬆️ | Upgrade dependency |
| ⬇️ | Downgrade dependency |
| 🔒 | Security fix |
| 🚑 | Critical hotfix |
| 💄 | UI or style change |
| 🏗️ | Architectural change |
| 🚧 | Work in progress |
| 🩹 | Simple or trivial fix |
| 💡 | Add or update comments |
| 🌐 | Internationalization |
| 🔀 | Merge branches |
| 🏷️ | Add or update types |

## Rules

- One line per commit. No body, no footer, no bullet points.
- Do not add any attribution, signature, watermark, or co-authorship line (no `Co-Authored-By`).
- Do not explain your reasoning. Just output the commit message(s).
