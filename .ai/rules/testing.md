# Testing Rules

## TDD Workflow (Red-Green-Refactor)
1. **Red:** Write a failing test first
2. **Green:** Write minimal code to make it pass
3. **Refactor:** Clean up code without changing behavior
4. **Commit:** After each cycle completes

**Why TDD:**
- Tests define behavior (spec)
- Prevents over-engineering
- Ensures testability
- Catches regressions early

## Test Coverage Requirements
- **All public functions** must have tests
- **Edge cases to test:**
  - Empty input
  - Nil values
  - Error conditions
  - Boundary values (0, 1, max)
- **Integration tests** for full task execution flows
- **No private function tests** — test through public API

## Test Naming
**Format:** `Test<FunctionName>_<Scenario>_<ExpectedBehavior>`

**Examples:**
- `TestLoad_FileNotFound_NoError`
- `TestLoad_ValidEnv_LoadsVariables`
- `TestLoad_WithLocal_OverridesBase`
- `TestLoad_WorldReadable_ReturnsError`

**Why this format:**
- Clear what's being tested
- Clear what the input scenario is
- Clear what the expected outcome is
- Easy to scan test list

## Test Structure (AAA Pattern)
```go
func TestLoad_ValidEnv_LoadsVariables(t *testing.T) {
    // Arrange - set up test data
    tmpDir := t.TempDir()
    envFile := filepath.Join(tmpDir, ".env")
    os.WriteFile(envFile, []byte("KEY=value"), 0600)
    
    // Act - execute the code under test
    err := Load(tmpDir)
    
    // Assert - verify the outcome
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if got := os.Getenv("KEY"); got != "value" {
        t.Errorf("got %q, want %q", got, "value")
    }
}
```

## Table-Driven Tests
Use for multiple similar test cases:
```go
func TestParseValue(t *testing.T) {
    tests := []struct {
        name  string
        input string
        want  string
    }{
        {"unquoted", "hello", "hello"},
        {"quoted", `"hello"`, "hello"},
        {"empty", "", ""},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := ParseValue(tt.input)
            if got != tt.want {
                t.Errorf("got %q, want %q", got, tt.want)
            }
        })
    }
}
```

## Test Helpers
- Use `t.Helper()` in helper functions:
  ```go
  func createEnvFile(t *testing.T, content string) string {
      t.Helper()
      tmpDir := t.TempDir()
      envFile := filepath.Join(tmpDir, ".env")
      if err := os.WriteFile(envFile, []byte(content), 0600); err != nil {
          t.Fatal(err)
      }
      return tmpDir
  }
  ```

## Temporary Files & Cleanup
- **Use `t.TempDir()`** for temp directories (auto-cleanup)
- **Use `t.Cleanup()`** for custom cleanup:
  ```go
  oldEnv := os.Getenv("KEY")
  os.Setenv("KEY", "test")
  t.Cleanup(func() {
      os.Setenv("KEY", oldEnv)
  })
  ```

## Integration Tests
- Place in `cmd/xc/*_test.go`
- Test full flows: parse → run → verify
- Use real task files (or temp files with real content)
- Verify both stdout and exit codes

## Running Tests
```bash
# All tests
go test ./...

# Specific package
go test ./run

# Specific test
go test ./run -run TestLoad_ValidEnv

# With coverage
go test -cover ./...

# With race detector
go test -race ./...

# Verbose
go test -v ./...
```

## Test Data
- **Inline test data** for small cases
- **Separate test files** for complex cases (in `testdata/` directory)
- **Never commit .env files** with secrets (use dummy values)

## What NOT to Test
- Third-party library behavior (assume libraries work)
- Standard library functions
- Trivial getters/setters

## When Tests Fail
1. **Read the error message** — it should tell you what's wrong
2. **Check your assumptions** — is the test correct?
3. **Add debug output** if needed (`t.Logf("got: %v", value)`)
4. **Don't comment out tests** — fix or delete them
