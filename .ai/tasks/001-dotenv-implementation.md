# Task: Implement .env File Support

**ADR:** [001-dotenv-support.md](../architecture/adrs/001-dotenv-support.md)  
**Issue:** #162  
**Owner:** Thiago Pacheco  
**Started:** 2026-04-05  
**Status:** In Progress

---

## Goal
Add `.env` file loading to xc per ADR-001, following TDD principles and keeping commits small and atomic.

---

## Implementation Plan (TDD Order)

### Phase 1: Add Dependency
**Goal:** Add godotenv library to project

- [ ] **Task:** Add `github.com/joho/godotenv` to go.mod
  ```bash
  go get github.com/joho/godotenv
  go mod tidy
  ```
- [ ] **Verify:** `go list -m github.com/joho/godotenv` shows version
- [ ] **Commit:** `add godotenv dependency`

**Estimated time:** 1 minute  
**Risk:** Low (just dependency addition)

---

### Phase 2: Implement .env Loader (TDD)
**Goal:** Create a new package to handle .env loading with security checks

**Package location:** `internal/dotenv/` (new package)

#### Step 2.1: Test - File Not Found Is OK
- [ ] **Create:** `internal/dotenv/loader_test.go`
- [ ] **Write test:** `TestLoad_FileNotFound_NoError`
  ```go
  func TestLoad_FileNotFound_NoError(t *testing.T) {
      tmpDir := t.TempDir()
      err := Load(tmpDir)
      if err != nil {
          t.Errorf("expected no error, got %v", err)
      }
  }
  ```
- [ ] **Run test:** Should fail (function doesn't exist yet) ❌ RED
- [ ] **Implement:** `internal/dotenv/loader.go` with minimal `Load()` function
  ```go
  func Load(dir string) error {
      return nil  // Minimal implementation
  }
  ```
- [ ] **Run test:** Should pass ✅ GREEN
- [ ] **Refactor:** (none needed yet)
- [ ] **Commit:** `add dotenv loader with file not found handling`

**Estimated time:** 5 minutes

#### Step 2.2: Test - Valid .env Loads Variables
- [ ] **Write test:** `TestLoad_ValidEnv_LoadsVariables`
  ```go
  func TestLoad_ValidEnv_LoadsVariables(t *testing.T) {
      // Arrange
      tmpDir := t.TempDir()
      envFile := filepath.Join(tmpDir, ".env")
      content := "TEST_KEY=test_value\nANOTHER=value2"
      os.WriteFile(envFile, []byte(content), 0600)
      
      // Act
      err := Load(tmpDir)
      
      // Assert
      if err != nil {
          t.Fatalf("unexpected error: %v", err)
      }
      if got := os.Getenv("TEST_KEY"); got != "test_value" {
          t.Errorf("TEST_KEY = %q, want %q", got, "test_value")
      }
      if got := os.Getenv("ANOTHER"); got != "value2" {
          t.Errorf("ANOTHER = %q, want %q", got, "value2")
      }
      
      // Cleanup
      t.Cleanup(func() {
          os.Unsetenv("TEST_KEY")
          os.Unsetenv("ANOTHER")
      })
  }
  ```
- [ ] **Run test:** Should fail ❌ RED (Load() doesn't actually load)
- [ ] **Implement:** Use godotenv to load .env
  ```go
  func Load(dir string) error {
      envPath := filepath.Join(dir, ".env")
      if _, err := os.Stat(envPath); errors.Is(err, os.ErrNotExist) {
          return nil // File not found is OK
      }
      return godotenv.Load(envPath)
  }
  ```
- [ ] **Run test:** Should pass ✅ GREEN
- [ ] **Refactor:** Extract constants if needed
- [ ] **Commit:** `load env vars from dotenv file`

**Estimated time:** 10 minutes

#### Step 2.3: Test - .env.local Overrides .env
- [ ] **Write test:** `TestLoad_WithLocal_OverridesBase`
  ```go
  func TestLoad_WithLocal_OverridesBase(t *testing.T) {
      // Arrange
      tmpDir := t.TempDir()
      
      // Create .env with base values
      envFile := filepath.Join(tmpDir, ".env")
      os.WriteFile(envFile, []byte("KEY=base\nONLY_BASE=base_value"), 0600)
      
      // Create .env.local with overrides
      localFile := filepath.Join(tmpDir, ".env.local")
      os.WriteFile(localFile, []byte("KEY=local\nONLY_LOCAL=local_value"), 0600)
      
      // Act
      err := Load(tmpDir)
      
      // Assert
      if err != nil {
          t.Fatalf("unexpected error: %v", err)
      }
      if got := os.Getenv("KEY"); got != "local" {
          t.Errorf("KEY = %q, want %q (should be overridden)", got, "local")
      }
      if got := os.Getenv("ONLY_BASE"); got != "base_value" {
          t.Errorf("ONLY_BASE = %q, want %q", got, "base_value")
      }
      if got := os.Getenv("ONLY_LOCAL"); got != "local_value" {
          t.Errorf("ONLY_LOCAL = %q, want %q", got, "local_value")
      }
      
      // Cleanup
      t.Cleanup(func() {
          os.Unsetenv("KEY")
          os.Unsetenv("ONLY_BASE")
          os.Unsetenv("ONLY_LOCAL")
      })
  }
  ```
- [ ] **Run test:** Should fail ❌ RED (.env.local not loaded)
- [ ] **Implement:** Load both files
  ```go
  func Load(dir string) error {
      // Load .env
      envPath := filepath.Join(dir, ".env")
      if _, err := os.Stat(envPath); !errors.Is(err, os.ErrNotExist) {
          if err := godotenv.Load(envPath); err != nil {
              return err
          }
      }
      
      // Load .env.local (overrides)
      localPath := filepath.Join(dir, ".env.local")
      if _, err := os.Stat(localPath); !errors.Is(err, os.ErrNotExist) {
          if err := godotenv.Load(localPath); err != nil {
              return err
          }
      }
      
      return nil
  }
  ```
- [ ] **Run test:** Should pass ✅ GREEN
- [ ] **Refactor:** Extract helper function for loading single file
- [ ] **Commit:** `support dotenv local overrides`

**Estimated time:** 10 minutes

#### Step 2.4: Test - Security Check for World-Readable Files
- [ ] **Write test:** `TestLoad_WorldReadable_ReturnsWarning`
  ```go
  func TestLoad_WorldReadable_LogsWarning(t *testing.T) {
      // Arrange
      tmpDir := t.TempDir()
      envFile := filepath.Join(tmpDir, ".env")
      os.WriteFile(envFile, []byte("SECRET=exposed"), 0644) // World-readable!
      
      // Capture log output
      var logBuf bytes.Buffer
      log.SetOutput(&logBuf)
      t.Cleanup(func() {
          log.SetOutput(os.Stderr)
      })
      
      // Act
      err := Load(tmpDir)
      
      // Assert
      if err != nil {
          t.Fatalf("unexpected error: %v", err)
      }
      logOutput := logBuf.String()
      if !strings.Contains(logOutput, "world-readable") {
          t.Errorf("expected warning about world-readable, got: %s", logOutput)
      }
      // Verify SECRET was NOT loaded
      if got := os.Getenv("SECRET"); got != "" {
          t.Errorf("SECRET should not be loaded from world-readable file, got %q", got)
      }
  }
  ```
- [ ] **Run test:** Should fail ❌ RED (no security check yet)
- [ ] **Implement:** Add permission check
  ```go
  func loadFile(path string) error {
      info, err := os.Stat(path)
      if errors.Is(err, os.ErrNotExist) {
          return nil
      }
      if err != nil {
          return err
      }
      
      // Check permissions (Unix-specific, skip on Windows)
      if runtime.GOOS != "windows" {
          perm := info.Mode().Perm()
          if perm&0044 != 0 { // World or group readable
              log.Printf("warning: %s is world/group readable, skipping for security", path)
              return nil
          }
      }
      
      return godotenv.Load(path)
  }
  ```
- [ ] **Run test:** Should pass ✅ GREEN
- [ ] **Refactor:** Clean up permission logic
- [ ] **Commit:** `add security check for world readable dotenv`

**Estimated time:** 15 minutes

---

### Phase 3: Integrate into Main
**Goal:** Call dotenv.Load() at application startup

#### Step 3.1: Integration Point
- [ ] **Modify:** `cmd/xc/main.go`
- [ ] **Add import:** `"github.com/joerdav/xc/internal/dotenv"`
- [ ] **Add call:** In `runMain()`, before task parsing:
  ```go
  func runMain() error {
      // Load .env files
      cwd, err := os.Getwd()
      if err != nil {
          return fmt.Errorf("failed to get current directory: %w", err)
      }
      if err := dotenv.Load(cwd); err != nil {
          log.Printf("warning: failed to load .env: %v", err)
      }
      
      // ... rest of existing code
  }
  ```
- [ ] **Test manually:**
  ```bash
  cd /tmp/xc
  echo "TEST_VAR=hello" > .env
  go build ./cmd/xc
  ./xc -s  # Should list tasks with TEST_VAR available
  ```
- [ ] **Commit:** `integrate dotenv loading into main`

**Estimated time:** 5 minutes

---

### Phase 4: Add CLI Flags
**Goal:** Support `--env-file` and `--no-env` flags

#### Step 4.1: Add Flags to Config
- [ ] **Modify:** `cmd/xc/main.go` in `flags()` function
- [ ] **Add to config struct:**
  ```go
  type config struct {
      // ... existing fields
      noEnv   bool
      envFile string
  }
  ```
- [ ] **Add flag definitions:**
  ```go
  flag.BoolVar(&cfg.noEnv, "no-env", false, "skip loading .env files")
  flag.StringVar(&cfg.envFile, "env-file", ".env", "specify env file to load")
  ```
- [ ] **Update usage.txt:** Add flag documentation
- [ ] **Commit:** `add env file cli flags`

**Estimated time:** 5 minutes

#### Step 4.2: Use Flags in Load Logic
- [ ] **Modify:** dotenv.Load() call in `runMain()`
  ```go
  if !cfg.noEnv {
      cwd, _ := os.Getwd()
      // Use custom env file if specified
      if cfg.envFile != ".env" {
          if err := dotenv.LoadFile(filepath.Join(cwd, cfg.envFile)); err != nil {
              log.Printf("warning: failed to load %s: %v", cfg.envFile, err)
          }
      } else {
          // Default: load .env + .env.local
          if err := dotenv.Load(cwd); err != nil {
              log.Printf("warning: failed to load .env: %v", err)
          }
      }
  }
  ```
- [ ] **Add to dotenv package:** `LoadFile(path string)` helper
- [ ] **Test manually:**
  ```bash
  xc --no-env task        # Should skip .env
  xc --env-file .env.prod task  # Should load custom file
  ```
- [ ] **Commit:** `implement env file flag behavior`

**Estimated time:** 10 minutes

---

### Phase 5: Documentation
**Goal:** Update README and add examples

#### Step 5.1: Update README.md
- [ ] **Add section:** "Environment Variables"
  - Explain .env support
  - Show load order
  - Show .env.local pattern
  - Show CLI flags
- [ ] **Add examples:**
  - Basic .env file
  - .env.local override
  - Using --env-file flag
- [ ] **Update features list:** Add ".env file support" bullet
- [ ] **Commit:** `document dotenv support in readme`

**Estimated time:** 10 minutes

#### Step 5.2: Add Example .env File
- [ ] **Create:** `.env.example` in repo root
  ```
  # Example environment variables for xc tasks
  # Copy this to .env and customize
  
  # Example: API keys
  API_KEY=your_key_here
  
  # Example: Database connection
  DATABASE_URL=postgres://localhost/mydb
  
  # Example: Environment
  ENV=development
  ```
- [ ] **Update .gitignore:** Ensure `.env` and `.env.local` are ignored
- [ ] **Commit:** `add env file example`

**Estimated time:** 5 minutes

---

### Phase 6: Manual Testing & Verification
**Goal:** Verify feature works end-to-end

#### Test Cases
- [ ] **Test 1:** .env loads automatically
  ```bash
  echo "MY_VAR=test123" > .env
  cat > README.md << 'EOF'
  ## test
  ```
  echo "MY_VAR is: $MY_VAR"
  ```
  EOF
  xc test  # Should print "MY_VAR is: test123"
  ```

- [ ] **Test 2:** .env.local overrides .env
  ```bash
  echo "MY_VAR=base" > .env
  echo "MY_VAR=local" > .env.local
  xc test  # Should print "MY_VAR is: local"
  ```

- [ ] **Test 3:** --no-env skips loading
  ```bash
  xc --no-env test  # Should print "MY_VAR is: " (empty)
  ```

- [ ] **Test 4:** --env-file loads custom file
  ```bash
  echo "MY_VAR=prod" > .env.prod
  xc --env-file .env.prod test  # Should print "MY_VAR is: prod"
  ```

- [ ] **Test 5:** World-readable .env shows warning
  ```bash
  chmod 644 .env
  xc test 2>&1 | grep "world-readable"  # Should show warning
  ```

- [ ] **Test 6:** Task-level Env still works
  ```markdown
  ## test
  Env: MY_VAR=override
  ```
  echo $MY_VAR
  ```
  ```
  Should show task value overriding .env

- [ ] **Document test results:** In this file

**Estimated time:** 15 minutes

---

### Phase 7: Final Review & Polish
**Goal:** Ensure code quality and completeness

- [ ] **Run full test suite:** `go test ./...` (all pass)
- [ ] **Run linter:** `golangci-lint run` (no errors)
- [ ] **Run vet:** `go vet ./...` (no issues)
- [ ] **Check test coverage:** `go test -cover ./...` (>80% for dotenv package)
- [ ] **Review all commits:** Clean, atomic, good messages
- [ ] **Update this task file:** Mark all checkboxes complete

**Estimated time:** 10 minutes

---

## Success Criteria (Definition of Done)
- [ ] All tests passing (`go test ./...`)
- [ ] Linter clean (`golangci-lint run`)
- [ ] Feature works as designed (manual tests pass)
- [ ] Documentation complete (README.md has .env section)
- [ ] Example .env file in repo
- [ ] Commit history is clean (8-12 atomic commits)
- [ ] `.ai/` folder complete (this task marked done)

---

## Estimated Total Time
**Phases 1-7:** ~90-100 minutes (~1.5 hours)

**Breakdown:**
- Phase 1: 1 min
- Phase 2: 40 min (TDD cycles)
- Phase 3: 5 min
- Phase 4: 15 min
- Phase 5: 15 min
- Phase 6: 15 min
- Phase 7: 10 min

---

## Risk Assessment
**Low risk:**
- Using battle-tested library (godotenv)
- Backward compatible (doesn't break existing behavior)
- Graceful degradation (missing files are OK)

**Potential issues:**
- Permission checking on Windows (mitigated: skip on Windows)
- Load order confusion (mitigated: documented clearly)
- Existing env vars not being overridden (mitigated: documented load order)

---

## Next Steps After Completion
1. **Test on real project** (use xc with .env in a real workflow)
2. **Gather feedback** (does it match user expectations?)
3. **Consider PR upstream** (if maintainer is interested)
4. **Write blog post / video** (document the process)

---

## Notes & Learnings
*(To be filled in during implementation)*

- Learned: 
- Challenges:
- Improvements for next time:
