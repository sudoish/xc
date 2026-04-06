# Agents & Roles

## Project Maintainer
- **Name:** Thiago Pacheco (@sudoish)
- **Responsible for:** Architecture decisions, feature planning, code review
- **Tools:** OpenCode, Go toolchain, GitHub
- **Context:** This is a learning exercise in AI-driven development with structured context

## AI Pair Programmer
- **Role:** Code generation, refactoring suggestions, test writing, codebase exploration
- **Context sources:** This `.ai/` folder, codebase, commit history, GitHub issues
- **Constraints:**
  - Follow project rules in `.ai/rules/`
  - Write tests first (TDD workflow)
  - Keep commits small and atomic
  - One logical change per commit
  - Read the relevant ADR before implementing anything

## Communication Preferences
- **Commit messages:** One sentence, lowercase, no special characters (see `rules/commits.md`)
- **Testing:** TDD always — test first, code second
- **Documentation:** Update as you go, not after
- **Questions:** Ask when uncertain — don't guess at architecture
