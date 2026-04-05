# Architecture Decisions

## Current Architecture Overview

### Task Definition Language
**Decision:** Tasks are defined in Markdown (or Org-mode) documentation files.

**Rationale:** 
- Documentation and implementation live together (literate programming)
- No separate scripting language to learn
- Human-readable without special tools
- Familiar format (Markdown)

### Task Parsing
**Process:** Documentation → AST → Task structs

**Components:**
- Headings (## Task Name) become task names
- Text paragraphs become descriptions
- Special lines become metadata:
  - `Requires: task1, task2` → dependencies
  - `Inputs: VAR1, VAR2` → required inputs
  - `Directory: path` → working directory
  - `Env: KEY=VALUE` → environment variables
  - `Run: once|always` → execution behavior
  - `RunDeps: sync|async` → dependency execution mode
- Code blocks (```) become scripts

**Why this approach:**
- Self-documenting (the file IS the documentation)
- No learning curve (everyone knows Markdown)
- Extensible (new metadata can be added as plain text)

### Task Execution
**Process:** Dependency resolution → Environment setup → Script execution

**Environment merging order:**
1. System environment (`os.Environ()`)
2. Task-level env (`Env: KEY=VALUE` from task definition)
3. Input values (from CLI args or env)
4. Inline exports in script (normal bash behavior)

**Why this order:**
- System env provides baseline
- Task env provides defaults
- Inputs override defaults (explicit > implicit)
- Script exports are most specific (local scope)

### Dependency Handling
**Strategy:** Topological sort with cycle detection

**Behaviors:**
- `Run: always` (default) → Run every time required
- `Run: once` → Run once, skip on subsequent requires
- `RunDeps: sync` (default) → Run deps sequentially
- `RunDeps: async` → Run deps in parallel

**Constraints:**
- Max dependency depth: 50 (prevents infinite recursion)
- No cycles allowed (validated at parse time)

**Why these constraints:**
- Prevents common mistakes (circular dependencies)
- Predictable execution order
- Performance optimization (parallel when safe)

### File Discovery
**Strategy:** Search upward from current directory

**Order:**
1. Look for explicit file (`-f <filename>`)
2. Look in current directory:
   - `README.md`, `.README.md` (Markdown)
   - `README.org`, `.README.org` (Org-mode)
3. Look in parent directories (up to filesystem root)

**Why upward search:**
- Mimics git behavior (users expect this)
- Allows running tasks from subdirectories
- Single source of truth at project root

### CLI Design
**Philosophy:** Discoverability over brevity

**Pattern:** `xc [flags] [task] [inputs...]`

**Flags:**
- Both short (`-f`) and long (`--file`) forms
- Help accessible via `-h` or `--help`
- Version accessible via `-v` or `--version`

**Why this approach:**
- Familiar to users of make, npm run, etc.
- Self-documenting (help text shows all options)
- Progressive disclosure (simple usage is simple)

---

## Limitations & Known Issues

### No Configuration File
**Current state:** No `.xcrc` or `xc.toml` support

**Why:**
- Keeping it simple
- Tasks definitions ARE the configuration
- Avoids configuration sprawl

**Future consideration:**
- May add for global settings (default file paths, theme, etc.)

### No .env File Support (Addressing in ADR-001)
**Current state:** Environment must be exported externally or defined inline

**Impact:**
- Cluttered task definitions (`Env: KEY1=VAL1, KEY2=VAL2, ...`)
- Hard to switch environments (dev, staging, prod)
- Doesn't match common patterns (docker-compose, npm, etc.)

**Solution:** See ADR-001 for .env support design

### No Task Import/Include
**Current state:** Tasks are file-local, no importing from other files

**Why:**
- Simpler mental model
- Prevents dependency hell
- Encourages self-contained documentation

**Workaround:**
- Use shell scripts and call them from tasks
- Duplicate common tasks in each file

### No Task Templating
**Current state:** No way to parameterize task definitions

**Example of what's NOT possible:**
```markdown
## deploy-{env}  <!-- This doesn't work -->
Deploy to {env} environment.
```

**Why:**
- Adds complexity
- Markdown is static
- Inputs provide similar functionality

---

## Design Principles

### 1. Convention over Configuration
- Default behavior should "just work"
- Minimal flags required for common cases
- Explicit overrides available when needed

### 2. Fail Fast
- Validate dependencies at parse time
- Check required inputs before execution
- Clear error messages

### 3. Explicit over Implicit
- Task dependencies stated clearly
- Inputs declared upfront
- Environment handling is visible

### 4. Stay Close to the Shell
- Scripts are just shell scripts
- No DSL to learn
- Debugging is familiar (bash/cmd knowledge transfers)

### 5. Human-Readable
- Documentation IS the source code
- Non-technical users can read task files
- No special tools required to understand

---

## Future Considerations

### Potential Features (Not Planned Yet)
- Configuration file support
- Task templates/macros
- Remote task files (HTTP URLs)
- Task caching/memoization
- Watch mode (re-run on file changes)
- Multi-file task definitions

### Backward Compatibility Promise
- New features must not break existing task files
- Deprecations will have long sunset periods
- Metadata additions are additive only

---

## Related Documents
- ADR-001: .env File Support (in progress)
