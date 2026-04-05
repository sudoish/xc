# ADR-001: .env File Support

**Status:** Accepted  
**Date:** 2026-04-05  
**Deciders:** Thiago Pacheco  
**Issue:** [#162](https://github.com/joerdav/xc/issues/162)  
**Reporter:** zroadhouse

---

## Context

Users want to define environment variables in `.env` files instead of:
1. Exporting them manually before running `xc`
2. Defining them inline in task definitions (`Env: KEY1=VAL1, KEY2=VAL2`)

**Problem:**
- Inline `Env:` statements clutter Markdown
- Hard to switch between environments (dev, prod, staging)
- Doesn't match ecosystem patterns (docker-compose, npm scripts, etc. all use .env)
- Secrets management is awkward (env vars in documentation)

**User request:**
```markdown
## deploy
Deploy the application.

Env: DATABASE_URL=postgres://localhost, API_KEY=secret123, ENV=dev
```

Becomes:
```markdown
## deploy
Deploy the application.
```

With `.env` file:
```
DATABASE_URL=postgres://localhost
API_KEY=secret123
ENV=dev
```

---

## Decision

We will add `.env` file support with the following behavior:

### 1. File Discovery
- Look for `.env` in the **current working directory** (where xc is invoked)
- If `.env.local` exists, load it as well (overrides `.env`)
- `.env` is optional (missing file is not an error)
- Custom path supported via `--env-file` flag

**Rationale:**
- Current directory matches user expectation (same as docker-compose)
- `.env.local` pattern is standard (for local secrets, git-ignored)
- Optional behavior prevents breaking existing workflows

### 2. Load Order (Later Overrides Earlier)
1. System environment variables (`os.Environ()`)
2. `.env` file
3. `.env.local` file (if exists)
4. Task-level `Env:` statements
5. Input values (from CLI args)
6. Inline `export` statements in scripts

**Rationale:**
- Preserves existing behavior (task env and inputs still work)
- `.env` provides defaults
- `.env.local` provides local overrides (developer-specific)
- System env can still override everything (explicit user intent)

### 3. Security
- **Skip** `.env` files with world-readable permissions (chmod 644, 666)
- Log a **warning** if skipped: `warning: .env is world-readable, skipping for security`
- Only load if permissions are `600` (user read/write only) or `400` (user read only)

**Rationale:**
- Prevents accidental secret exposure
- Matches best practices (dotenv files should not be publicly readable)
- Doesn't break workflows (just warns and continues)

### 4. Error Handling
- **Missing .env:** Not an error (silent, feature is optional)
- **Malformed .env:** Log warning, skip bad lines, continue with rest
- **Permission error:** Log warning, skip file, continue

**Rationale:**
- Graceful degradation (don't break workflows)
- Users can run xc without .env files (no forced dependency)
- Clear feedback when something is wrong (warnings)

### 5. Implementation Details
- **Library:** Use `github.com/joho/godotenv` (standard Go .env library)
  - Battle-tested (10k+ stars)
  - Handles edge cases (quotes, escaping, comments)
  - Small, no dependencies
- **Loading point:** In `cmd/xc/main.go`, **before** task parsing
- **Environment mutation:** Use `godotenv.Load()` which mutates `os.Environ()`

**Rationale:**
- Don't reinvent .env parsing (complex edge cases)
- Load early so env is available during task parsing
- Direct mutation is simplest (no need to thread env through entire codebase)

### 6. CLI Flags
```
--env-file <path>    Load environment from specified file (default: .env)
--no-env             Skip loading .env files entirely
```

**Examples:**
```bash
# Default behavior (load .env and .env.local)
xc deploy

# Custom env file
xc --env-file .env.prod deploy

# Skip .env loading
xc --no-env deploy

# Multiple custom files (not supported initially)
# (Future: --env-file could be repeatable)
```

**Rationale:**
- `--env-file` provides escape hatch (custom environments)
- `--no-env` allows opting out (if .env conflicts with system env)
- Default behavior "just works" for 90% of users

---

## Alternatives Considered

### Option 1: Load .env at Startup (CHOSEN)
**Pros:**
- Simple, predictable
- Matches user expectations (docker-compose, npm, etc.)
- Global env available to all tasks
- Easy to implement

**Cons:**
- All tasks see same environment (no per-task isolation)
- Can't have task-specific .env files

**Decision:** **Chosen** — Simplicity and predictability outweigh isolation concerns.

### Option 2: Load .env Per-Task
**Pros:**
- Task isolation (each task could have `.env.taskname`)
- More granular control

**Cons:**
- Surprising behavior (tasks see different vars)
- Complex to implement (per-task file discovery)
- Violates principle of least surprise

**Decision:** **Rejected** — Too complex, doesn't match ecosystem patterns.

### Option 3: Require Explicit Flag (No Default Loading)
**Pros:**
- User control (opt-in)
- Security (explicit is safer)
- No behavior change for existing users

**Cons:**
- Extra CLI friction (every invocation needs `--env-file`)
- Doesn't match ecosystem (most tools load .env by default)

**Decision:** **Rejected** — Convenience matters, warnings handle security.

### Option 4: Configuration File (.xcrc)
**Pros:**
- Could configure env loading behavior
- Global settings across projects

**Cons:**
- Adds complexity (another file to learn)
- Configuration sprawl
- Doesn't solve immediate problem

**Decision:** **Rejected** — Out of scope, can revisit later.

---

## Consequences

### Positive
✅ Cleaner Markdown (no env clutter)  
✅ Easy environment switching (.env.dev, .env.prod)  
✅ Matches ecosystem patterns (familiar to all developers)  
✅ Supports local overrides (.env.local for secrets)  
✅ Security warning prevents common mistakes  
✅ Backward compatible (doesn't break existing tasks)

### Negative
⚠️ All tasks see same environment (no per-task isolation)  
⚠️ `.env.local` pattern might be unfamiliar to some users  
⚠️ Adds dependency (godotenv library)

### Neutral
- System env can still override .env (some users expect opposite)
- Missing .env is silent (might surprise users expecting error)

### Mitigation
- **Document load order clearly** in README (with examples)
- **Show .env + .env.local usage** in examples
- **Add warning for world-readable files** (educate users)
- **Provide --no-env flag** (escape hatch)

---

## Implementation Checklist
See `.ai/tasks/001-dotenv-implementation.md` for detailed plan.

- [ ] Add godotenv dependency
- [ ] Implement loader function (with security checks)
- [ ] Integrate into main.go
- [ ] Add CLI flags (--env-file, --no-env)
- [ ] Write tests (TDD)
- [ ] Update documentation (README.md)
- [ ] Add example .env file
- [ ] Manual testing

---

## Future Enhancements (Not in Scope)

### Possible Future Work
- Support multiple .env files (`--env-file` repeatable)
- Variable expansion (e.g., `PATH=$PATH:/usr/local/bin`)
- .env templates (.env.example → .env)
- Encrypted .env files (via external tool)
- Per-task .env files (.env.taskname)

### Won't Do
- Custom .env parsers (use godotenv)
- Complex variable interpolation (use shell for that)
- Automatic .env.example generation (out of scope)

---

## References
- GitHub Issue: https://github.com/joerdav/xc/issues/162
- godotenv library: https://github.com/joho/godotenv
- Twelve-Factor App (Environment Config): https://12factor.net/config
- docker-compose .env docs: https://docs.docker.com/compose/environment-variables/

---

## Approval
**Approved by:** Thiago Pacheco  
**Date:** 2026-04-05  
**Next step:** Implementation (see tasks/001-dotenv-implementation.md)
