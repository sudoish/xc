# xc Project Context

## What is xc?
Markdown-defined task runner. Parses code blocks in README.md (or any .md / .org file) and executes them as tasks.

**Philosophy:** Tasks should live inline with their documentation (literate programming).

## Architecture

### Core Packages
- `cmd/xc/` — CLI entry point, flag parsing, task selection
- `models/` — Core data structures (Task, RequiredBehaviour, DepsBehaviour)
- `parser/` — Task extraction from documentation
  - `parsemd/` — Markdown parser
  - `parseorg/` — Org-mode parser  
- `run/` — Task execution engine

### Data Model (`models/models.go`)
```go
type Task struct {
    Name              string
    Description       []string
    Script            string
    Dir               string      // Working directory for task
    Env               []string    // Environment variables (KEY=VALUE format)
    DependsOn         []string    // Task dependencies
    Inputs            []string    // Required inputs (from CLI args or env)
    RequiredBehaviour RequiredBehaviour  // once | always
    DepsBehaviour     DepsBehaviour      // sync | async
    Interactive       bool        // (deprecated)
}
```

### Task Parsing Flow
1. Search for documentation file (README.md, .README.md, etc.)
2. Parse headings as task names
3. Extract metadata from text (Requires:, Inputs:, Directory:, Env:, Run:, RunDeps:)
4. Extract code blocks as scripts
5. Return `models.Tasks` (slice of Task structs)

### Task Execution Flow (`run/run.go`)
1. **Dependency resolution:** Validate no cycles, max depth 50
2. **Run dependencies first:** Topological sort, respect RequiredBehaviour
3. **Prepare environment:**
   ```go
   env := os.Environ()                    // Start with system env
   for _, e := range task.Env {           // Layer task-specific env
       env = append(env, e)
   }
   ```
4. **Handle inputs:** CLI args or env vars → `KEY=VALUE` format
5. **Execute script:** Run via shell interpreter (bash on Unix, cmd on Windows)

### Key Behaviors
- **Dependency handling:** Tasks can require other tasks (`Requires: test, build`)
- **Input handling:** Tasks can require inputs (`Inputs: VERSION`)
  - Can be provided as CLI args: `xc deploy v1.0`
  - Or as environment vars: `VERSION=v1.0 xc deploy`
- **Environment merging:** Task `Env:` statements override system env
- **Directory context:** Tasks can specify working directory (`Directory: src`)
- **Trace mode:** `XC_TRACE=1` enables `set -o xtrace` in scripts

### Current Limitations
- **No .env file support** (Issue #162)
- Environment must be set externally or defined inline in tasks
- No configuration file support
- No global task definitions (tasks are per-file)

## File Discovery
xc searches upward from current directory for:
1. `README.md` or `.README.md` (Markdown)
2. `README.org` or `.README.org` (Org-mode)
3. Any file specified with `-f <filename>`

## Testing Strategy
- Unit tests in `*_test.go` files
- Table-driven tests where applicable
- Integration tests in `cmd/xc/` for end-to-end flows

## Build & Development
- **Build:** `go build ./cmd/xc`
- **Test:** `go test ./...`
- **Lint:** `golangci-lint run`
- **Release:** GoReleaser config (`.goreleaser.yml`)
