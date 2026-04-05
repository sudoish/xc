# Agents & Roles

## Project Maintainer
- **Name:** Thiago Pacheco (@sudoish)
- **Responsible for:** Architecture decisions, feature planning, code review
- **Tools:** OpenCode, Go toolchain, GitHub
- **Context:** This is a learning exercise in AI-driven development with structured context

## AI Pair Programmer (OpenCode + Free Model)
- **Role:** Code generation, refactoring suggestions, test writing, codebase exploration
- **Context sources:** 
  - `.ai/` folder (this directory)
  - Codebase (all `.go` files)
  - Commit history
  - GitHub issue #162
- **Constraints:** 
  - Follow project rules in `.ai/rules/`
  - Write tests first (TDD workflow)
  - Keep commits small and atomic
  - One logical change per commit

---

## Current Task: Add .env File Support

**GitHub Issue:** #162  
**Reporter:** zroadhouse  
**Owner:** Thiago Pacheco  
**Status:** In progress  
**Started:** 2026-04-05

### Task Summary
Add support for loading environment variables from `.env` files (and `.env.local` overrides) at application startup, providing an alternative to inline `Env:` statements in tasks.

**User value:**
- Cleaner Markdown (no environment clutter)
- Easy environment switching (dev, staging, prod)
- Standard .env pattern (familiar to all developers)

**Technical scope:**
- Add `.env` file loading at startup
- Support `.env.local` overrides
- Add `--env-file` CLI flag (optional custom path)
- Add `--no-env` flag to skip loading
- Security: skip world-readable files
- Tests for all scenarios
- Documentation updates

### Decision Log
See `.ai/architecture/adrs/001-dotenv-support.md` for full design decisions.

### Implementation Plan
See `.ai/tasks/001-dotenv-implementation.md` for step-by-step plan.

---

## Communication Preferences
- **Commit messages:** One sentence, lowercase, no special characters (see `.ai/rules/commits.md`)
- **Testing:** TDD always — test first, code second
- **Documentation:** Update as you go, not after
- **Questions:** Ask when uncertain — don't guess at architecture

## Success Criteria
- [ ] All tests pass (`go test ./...`)
- [ ] Linter clean (`golangci-lint run`)
- [ ] Feature works as designed (manual testing)
- [ ] Documentation complete (README.md updated)
- [ ] `.ai/` structure complete (for next contributor)
