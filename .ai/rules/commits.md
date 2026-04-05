# Commit Rules

## Commit Message Format
**Rule:** One sentence, lowercase, imperative mood, no special characters, max 72 chars

**Good examples:**
- `add godotenv dependency`
- `add dotenv loader with file not found handling`
- `load env vars from dotenv file`
- `support dotenv local overrides`
- `add security check for world readable dotenv`
- `integrate dotenv loading into main`
- `add no-env flag`
- `document dotenv support`

**Bad examples:**
- `Add godotenv dependency.` (capitalized, has period)
- `Added the godotenv library for .env file parsing` (past tense, too detailed)
- `WIP: dotenv stuff` (not descriptive)
- `Fix bug` (not specific enough)

## Why This Format?
- **Lowercase:** Consistent, easy to scan
- **Imperative mood:** "add X" not "added X" (matches git's own messages: "Merge branch...")
- **No special chars:** Clean git log, no markdown/emoji pollution
- **One sentence:** Forces clarity and focus
- **Max 72 chars:** Readable in `git log --oneline`

## Commit Size
**Rule:** One logical change per commit

**Examples of atomic commits:**
- Add a dependency
- Write one test
- Implement one function
- Update one doc section
- Fix one bug

**Too large:**
- Add feature, tests, and docs all in one commit
- Refactor multiple files at once
- Fix multiple bugs together

**Too small:**
- Fix typo in comment
- Add blank line for readability

**Balance:** Each commit should compile and pass tests (when possible)

## When to Commit
**TDD Cycle:**
1. Write test (red)
2. Write code (green)
3. Refactor (clean)
4. **Commit**

**General rule:** Commit after each complete thought

## What to Include
- **Code changes** + **tests** for that code (together)
- **Documentation** if it relates to this commit's change
- **Related files** only (don't fix unrelated things)

## Commit Frequency
**High frequency is good:**
- Easy to review
- Easy to revert
- Clear history
- Better bisect results

**If you find yourself writing "and"** in a commit message, it should probably be two commits:
- ❌ "add loader and integrate into main"
- ✅ "add dotenv loader" + "integrate dotenv into main" (two commits)

## Branch Strategy
**For this feature:**
- Work directly on fork's `main` branch (no feature branch needed)
- Can squash later if PR upstream

**For larger work:**
- Create feature branches
- Keep commits clean on branch
- Squash or rebase before merge (maintainer preference)

## Amending Commits
**OK to amend if:**
- Commit not pushed yet
- Fixing a typo or small mistake
- Adding forgotten file

**Command:** `git commit --amend --no-edit`

**NOT ok to amend if:**
- Commit already pushed (force push required)
- Working with others (confusing)

## Example Commit History (What We're Building)
```
* document dotenv support
* add no-env flag  
* integrate dotenv loading into main
* add security check for world readable dotenv
* support dotenv local overrides
* load env vars from dotenv file
* add dotenv loader with file not found handling
* add godotenv dependency
```

Clean, linear, easy to understand.

## Verification Before Commit
```bash
# Does it build?
go build ./cmd/xc

# Do tests pass?
go test ./...

# Lint clean?
golangci-lint run

# Stage only related files
git add <specific files>

# Commit
git commit -m "your message here"
```

## If You Make a Mistake
**Undo last commit (keep changes):**
```bash
git reset HEAD~1
```

**Undo last commit (discard changes):**
```bash
git reset --hard HEAD~1
```

**Change last commit message:**
```bash
git commit --amend
```

## Commit vs. Push
- **Commit often:** Local, cheap, safe
- **Push periodically:** After milestones, end of day, or when wanting backup
