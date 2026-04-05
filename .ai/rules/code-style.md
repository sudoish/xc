# Code Style Rules

## Go Conventions
- **Formatting:** Follow `gofmt` (run automatically on save in most editors)
- **Linting:** Use `golangci-lint run` (config in `.golangci.yaml`)
- **Function size:** Prefer small, focused functions (< 50 lines ideal)
- **Documentation:** Document all exported (public) functions, types, and packages
- **Package comments:** First comment in package should be `// Package <name> <description>.`

## Naming Conventions
- **Packages:** lowercase, single word (`models`, `run`, not `task_models`)
- **Files:** lowercase with underscores ok (`task_runner.go`)
- **Interfaces:** 
  - Single-method interfaces end in `-er` (`Runner`, `Parser`)
  - Multi-method interfaces are descriptive (`ScriptRunner`)
- **Variables:** 
  - Short names in small scopes (`i`, `t`, `err`)
  - Descriptive names in larger scopes (`taskName`, `environmentVars`)
- **Constants:** MixedCaps (not ALL_CAPS) — `MaxDeps`, not `MAX_DEPS`
- **Errors:** Exported errors prefixed with `Err` — `ErrNoTaskFile`

## Error Handling
- **Never panic** in library code — return errors
- **Wrap errors** with context using `fmt.Errorf`:
  ```go
  return fmt.Errorf("failed to parse task %q: %w", name, err)
  ```
- **Sentinel errors** should be exported: `var ErrNoTaskFile = errors.New("...")`
- **Check errors immediately** — don't defer checks

## Code Organization
- **Imports:** Grouped in order:
  1. Standard library
  2. External packages
  3. Internal packages
  
  Example:
  ```go
  import (
      "context"
      "fmt"
      
      "github.com/google/shlex"
      
      "github.com/joerdav/xc/models"
  )
  ```

## Testing
- **Test files:** Named `*_test.go` in same package
- **Test functions:** `Test<FunctionName>(t *testing.T)`
- **Table-driven tests** for multiple cases:
  ```go
  tests := []struct {
      name string
      input string
      want string
  }{
      {"empty", "", ""},
      {"simple", "hello", "HELLO"},
  }
  for _, tt := range tests {
      t.Run(tt.name, func(t *testing.T) {
          got := upper(tt.input)
          if got != tt.want {
              t.Errorf("got %q, want %q", got, tt.want)
          }
      })
  }
  ```

## Comments
- **When to comment:**
  - Public APIs (always)
  - Non-obvious logic
  - Workarounds or hacks (with explanation)
- **When not to comment:**
  - Obvious code (`i++ // increment i`)
  - Restating the code in English
- **TODO comments:** Include context
  ```go
  // TODO(username): reason why this is temporary
  ```

## Specific to xc
- **Task execution:** All env handling happens in `run/run.go`
- **Parsing:** Parser packages should be pure — no side effects
- **Errors:** User-facing errors should be clear and actionable
- **CLI flags:** Use both short (`-f`) and long (`--file`) forms
