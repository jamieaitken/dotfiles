# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Environment

- macOS ARM (Apple Silicon), Homebrew
- Go 1.25, Python 3.9
- IDE: GoLand

## Go Conventions

- **Unit tests**: Always use table-driven tests (`tests := []struct{...}` with `for _, tt := range tests`)
- **Context**: All I/O operations (HTTP calls, database queries, file operations, etc.) must accept and propagate `context.Context`
- **Cyclomatic complexity**: Functions must not exceed a cyclomatic complexity of 10. Break complex functions into smaller, focused helpers
- **Exported type comments**: All exported types must have a comment explaining their purpose (e.g., `// CD is responsible for foo.`)
- **Error equality**: Use `errors.Is` for error comparison instead of `==`
- **Interfaces**: Define interfaces where they are consumed, not where they are implemented. The consumer package owns the abstraction it depends on
- **Early returns (bouncing pattern)**: Use early returns (guard clauses) instead of `else` and `else if` blocks. Bounce out of the function as early as possible to minimise nesting and indentation within the function body

```go
// Preferred
if err != nil {
    return err
}
// continue with happy path

// Avoid
if err != nil {
    return err
} else {
    // happy path
}
```

- **Logging**: Use `log/slog` for all logging. Use `slog.InfoContext`, `slog.ErrorContext`, `slog.WarnContext` etc. with structured key-value attributes. Never use the `log` package or `fmt.Println` for logging
