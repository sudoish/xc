# xc — AI Development Context

This project uses a structured `.ai/` folder for AI-assisted development.
Read it before starting any work.

## Quick Start

1. **Understand the project** — Read `.ai/context.md`
2. **Check current architecture** — Read `.ai/architecture/decisions.md`
3. **Find the relevant ADR** — Check `.ai/architecture/adrs/` for design decisions
4. **Follow the rules** — `.ai/rules/` defines code style, testing, and commit conventions
5. **Check active work** — `.ai/tasks/` has implementation specs for in-progress features

## Workflow for New Features

1. Create a new ADR in `.ai/architecture/adrs/` (e.g., `002-feature-name.md`)
2. Create a task spec in `.ai/tasks/` (e.g., `002-feature-implementation.md`)
3. Update `.ai/architecture/decisions.md` if the feature changes project architecture
4. Follow TDD workflow defined in `.ai/rules/testing.md`
5. Commit conventions are in `.ai/rules/commits.md`

## Key Principles

- **ADR before code** — Design decisions are captured before implementation starts
- **TDD always** — Write the test first, implement second, refactor third
- **Small commits** — One logical change per commit, atomic and revertable
- **Documentation compounds** — Update `.ai/` docs as the project evolves

## References

- **Project context:** `.ai/context.md`
- **Architecture:** `.ai/architecture/decisions.md`
- **Design decisions:** `.ai/architecture/adrs/`
- **Code style:** `.ai/rules/code-style.md`
- **Testing rules:** `.ai/rules/testing.md`
- **Commit rules:** `.ai/rules/commits.md`
- **Active tasks:** `.ai/tasks/`
