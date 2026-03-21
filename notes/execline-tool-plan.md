# Execline Tool Implementation Plan

## Overview

Create a separate `ExeclineTool` for picoclaw that uses `execlineb` instead of `sh -c` for improved security through simplicity.

## Current State

- **Exec tool**: `/code/picoclaw/pkg/tools/shell.go` (~400 lines)
- **Execution**: Uses `sh -c` with extensive regex-based deny patterns
- **Config**: `ExecConfig` in `/code/picoclaw/pkg/config/config.go`

## Revised Approach: Start with Skill

Before implementing as a core tool, start with an **execline skill** to validate:
1. LLMs can generate valid execline syntax
2. Error messages are understandable
3. Use cases that benefit from execline

This aligns with picoclaw's skill-based extensibility.

## Goals

1. Create a separate tool (not modify existing exec)
2. Use execline for reduced attack surface
3. Leverage execline's design: no variable expansion, no command substitution by default

## Implementation

### 1. New Tool File

**Location**: `/code/picoclaw/pkg/tools/execline.go`

Reuse the environment sanitisation code from PR1261 (merged) that exists in `exec.go`:
- Working directory validation
- Path allowlisting
- Environment variable sanitisation

```go
package tools

// ExeclineTool implements a safe command executor using execlineb
type ExeclineTool struct {
    // Embed common tool fields
    BaseTool
    
    // Config
    AllowTimeout bool
    MaxSize      int64
}

// Execute runs the command via execlineb -c
func (t *ExeclineTool) Execute(ctx context.Context, input *ExecuteInput) (*ExecuteOutput, error)
```

### 2. Configuration

**File**: `/code/picoclaw/pkg/config/config.go`

Add to `ToolsConfig`:
```go
type ToolsConfig struct {
    // ... existing fields ...
    
    Execline struct {
        AllowTimeout bool
        MaxSize      int64
    }
}
```

### 3. Prompt/Instructions

The LLM is instructed to generate execline syntax directly. No translation layer needed.

**Key execline concepts to include in system prompt**:
- Use `foreground { cmd1 } { cmd2 }` instead of `cmd1 && cmd2`
- Use `ifelse { cmd1 } { cmd2 }` instead of `cmd1 || cmd2`
- Use `redirfd -w 1 file cmd` instead of `cmd > file`
- No variable expansion (`$VAR` won't work)
- No command substitution (backticks won't work)

### 4. Registration

**File**: `/code/picoclaw/pkg/tools/registry.go`

```go
func init() {
    Registry.Register(&ExeclineTool{
        Name:        "execline",
        Description: "Execute commands using execlineb",
    })
}
```

## Key Differences from Exec Tool

| Feature | exec | execline |
|---------|------|----------|
| Shell | `sh -c` | `execlineb -c` |
| Guard patterns | 40+ deny regexes | Minimal (path checks only) |
| Command substitution | Blocked by regex | Impossible by design |
| Variable expansion | Blocked by regex | Impossible by design |
| Redirections | Complex parsing | Native execline syntax |

## Risks & Mitigations

1. **Risk**: User provides execline syntax errors
   - **Mitigation**: Clear error messages from execlineb

2. **Risk**: Path traversal in commands
   - **Mitigation**: Validate working directory

3. **Risk**: execlineb not installed
   - **Mitigation**: Check availability, provide clear error

## Testing

1. Integration tests with known safe/unsafe inputs
2. Verify execlineb availability on target systems

## Test Strategy (Go Reimplementation Consideration)

If we ever consider reimplementing execline in pure Go, a comprehensive test suite would be needed:

### Test Categories

1. **Unit tests per command** - each builtin (if, for, foreground, etc.) tested in isolation
2. **Script equivalence tests** - compare Go implementation output against original execlineb
3. **Security tests** - verify no `$VAR` or `$(cmd)` expansion happens in quoted blocks
4. **Edge cases** - empty scripts, special chars, large scripts, nested blocks
5. **Integration tests** - full scripts from real use cases

### Practical Note

However, a pure Go implementation is likely unnecessary because:
- The original execlineb is tiny (~400KB), C-based, and already exists
- Implementing ~30 builtins natively in Go is a major effort
- We can simply wrap execlineb as a subprocess (which is what we're doing)

For our wrapper (`ExeclineTool`), we should test:
- **Golden tests**: Generated scripts match expected output
- **Integration**: Scripts execute correctly via execlineb
- **Security**: Blocklisted vars can't be overridden
- **Error handling**: Invalid scripts, missing execlineb, timeouts

## Dependencies

- `execline` package (available on most Linux systems)
- Already used in Alpine Linux, Armbian minimal images
