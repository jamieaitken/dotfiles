# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Environment

- macOS ARM (Apple Silicon), Homebrew
- Go 1.25, Python 3.9
- IDE: GoLand

## Go Conventions

### Project Structure & CLI

- **Project layout**: All Go projects must follow the [golang-standards/project-layout](https://github.com/golang-standards/project-layout) structure (e.g., `cmd/`, `internal/`, `pkg/`, `api/`, `configs/`, `scripts/`, etc.)
- **CLI entrypoints**: All Go applications must use [spf13/cobra](https://github.com/spf13/cobra) for their entrypoints. Each application has a single root command with subcommands for each concern (e.g., `serve` for the HTTP server, `mcp` for the MCP server, `migrate` for database migrations, etc.)
- **Cyclomatic complexity**: Functions must not exceed a cyclomatic complexity of 10. Break complex functions into smaller, focused helpers

### Formatting

- Run `goimports` on all code; never manually fix mechanical style issues
- Use tabs for indentation, not spaces
- No rigid line length limit; break lines for semantic reasons, not length. Wrap long lines with an extra tab indent
- Opening braces on the same line as the control structure
- Struct literals for types defined outside the current package must specify field names
- Closing brace of a multi-line literal goes on a line with the same indentation as the opening line, preceded by a trailing comma
- Do not break function/method signatures arbitrarily based on line length; keep on a single line when possible
- Comment lines should be wrapped at 80-100 columns; maintain consistency within a file

### Naming — General

- Use `MixedCaps` or `mixedCaps` for all multi-word identifiers; never use underscores or `ALL_CAPS`
- Initialisms/acronyms must have consistent case: `URL` or `url`, never `Url`; `ServeHTTP` not `ServeHttp`; `appID` not `appId`
- Constants use `MixedCaps`, never `UPPER_SNAKE_CASE`. Name constants based on their role, not their value
- Short variable names; length proportional to scope distance from declaration. `c` over `lineCount`, `i` over `sliceIndex`
- Single-letter names are fine for loop indices (`i`), readers (`r`), writers (`w`), clients (`c`)
- Do not drop letters to abbreviate: `Sandbox` not `Sbx`
- Omit type-related words from variable names: `users` not `userSlice`, `count` not `usersInt`
- Omit words already clear from context (package name, method name, type name)
- Use `any` instead of `interface{}` in new code
- In conditionals comparing a variable to a constant, place the variable on the left: `if x == 5` not `if 5 == x`

### Naming — Packages

- Package names must be lowercase, single-word, concise; no underscores or mixedCaps
- Do not use meaningless names: `util`, `common`, `misc`, `api`, `types`, `interfaces`, `helper`, `model`
- Exported identifiers must not repeat the package name: `chubby.File` not `chubby.ChubbyFile`
- Package comment starts with "Package [name]..." immediately above the `package` clause with no blank line
- For `package main`, use "Binary [name]...", "Command [name]...", or "The [name] command..."
- Exactly one file per package should contain the package comment; use `doc.go` for long comments

### Naming — Functions, Methods, Receivers

- Getter methods omit "Get": `Owner()` not `GetOwner()`. Use `Compute` or `Fetch` when the operation is expensive or performs remote calls
- Setter methods use the `Set` prefix: `SetOwner()`
- Single-method interfaces use method name + `-er` suffix: `Reader`, `Writer`, `Formatter`
- Reuse canonical method names (`Read`, `Write`, `Close`, `Flush`, `String`) only when semantics match exactly
- String converter: `String()` not `ToString()`
- Setup helpers that stop on failure use `MustXYZ` naming; only call at program startup, never in request handlers
- If a package exports one primary type named after the package, call the constructor `New` (e.g., `ring.New`)
- Receiver names: 1-2 letter abbreviation of the type (`c` for `Client`); never `me`, `this`, `self`
- Receiver name must be consistent across all methods of a type; never use underscore as a receiver name

### Naming — Result Parameters

- Do not name result parameters just to avoid declaring a variable or to enable naked returns
- Name result parameters when a function returns 2-3 parameters of the same type, or when meaning is unclear
- Consider how named result parameters appear in godoc

### Comments & Documentation

- **Exported type comments**: All exported types must have a comment explaining their purpose (e.g., `// CD is responsible for foo.`)
- Doc comments must be full sentences, begin with the name of the declared item, and end with a period
- All top-level exported names must have doc comments; non-trivial unexported types/functions should also have doc comments
- Include runnable `Example` functions in test files demonstrating usage of new packages

### Imports

- Organize imports in groups separated by blank lines: (1) stdlib, (2) external packages
- Do not rename imports except to avoid name collisions; rename the most local/project-specific import when collision occurs
- Blank imports (`import _ "pkg"`) only in main packages or tests, never in library packages
- Do not use `import .` except in tests with circular dependency issues

### Error Handling

- **Error equality**: Use `errors.Is` for error comparison instead of `==`
- Never discard errors using `_`; always check error returns. When ignoring in rare cases (e.g., `(*bytes.Buffer).Write`), add a comment explaining why
- Error strings must not be capitalized (unless proper nouns/acronyms) and must not end with punctuation
- Error strings should identify their origin with a prefix: `"image: unknown format"`
- Use multiple return values for errors; error should be the final return value
- Do not return in-band error values (`-1`, `""`, `nil`) to signal errors; use `(value, error)` or `(value, bool)`
- Do not use `panic` for normal error handling; reserve for truly unrecoverable situations. Libraries should never panic
- Exception: `panic` is acceptable during initialization if the library cannot set itself up
- Exported functions returning errors should return the `error` interface type, not concrete error types
- For `package main`, consider `log.Fatal` for fatal startup errors instead of `panic`

### Control Flow

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

- Indent error handling, keep the happy path at minimal indentation
- If `if` body ends with `break`, `continue`, `goto`, or `return`, omit the `else`
- Use initialization statements in `if`: `if err := f(); err != nil { ... }`. If the initialized variable is needed after the `if`, move declaration before the `if`
- Switch cases do not fall through by default; use `fallthrough` explicitly if needed. Do not use redundant `break`
- Use labeled `break` to exit a `for` loop from within a `switch`
- Use expression-less `switch` as an `if-else-if-else` chain
- Naked returns only in very short functions (a handful of lines); be explicit in medium+ functions

### Context

- **Context**: All I/O operations (HTTP calls, database queries, file operations, etc.) must accept and propagate `context.Context`
- `context.Context` must be the first parameter: `func F(ctx context.Context, ...)`
- Pass Context explicitly along the entire call chain; do not store it in a struct field
- Do not create custom Context types or use interfaces other than `context.Context`
- Only use `context.Background()` at entrypoints; code in the middle of a call chain must not create base contexts
- In tests, prefer `(testing.TB).Context()` over `context.Background()`

### Concurrency

- Make goroutine lifetimes obvious; document when/why they exit. Never start a goroutine without knowing how it will stop
- Prefer synchronous functions; callers can add concurrency by calling in a separate goroutine. Removing unwanted concurrency is hard
- Do not communicate by sharing memory; share memory by communicating (use channels)
- Use buffered channels as semaphores to limit throughput
- Use `select` with `default` for non-blocking channel operations

### Interfaces

- **Interfaces**: Define interfaces where they are consumed, not where they are implemented. The consumer package owns the abstraction it depends on
- Return concrete types from implementing packages; accept interfaces in consuming functions
- Do not define interfaces before they are used; wait until a concrete need arises
- Do not define interfaces "for mocking"; design APIs testable via public API of the real implementation
- Keep interfaces small (1-2 methods); they compose better
- Use `var _ json.Marshaler = (*MyType)(nil)` to verify interface compliance at compile time
- Use "comma ok" for safe type assertions: `v, ok := i.(string)` to avoid panics

### Data Types & Allocation

- Prefer `var t []string` (nil slice) over `t := []string{}` for declaring empty slices
- Use `len(s)` to check emptiness, not `s == nil`
- Do not design APIs that distinguish between nil and empty slices
- Design data structures so zero values are useful without initialization
- Use composite literals with field labels: `&File{fd: fd, name: name}`
- Do not copy structs containing `sync.Mutex` or similar synchronization fields
- Do not copy a value of type `T` if its methods are on `*T`
- Use `map[T]bool` for sets; test membership with `if m[x]`
- Use "comma ok" to distinguish missing map keys from zero values: `v, ok := m[key]`
- Always reassign the result of `append`: `x = append(x, elem)`

### Receiver Type (value vs pointer)

- When in doubt, use a pointer receiver
- Use pointer receiver if the method mutates the receiver, contains `sync.Mutex`, or the struct is large
- Use value receiver for maps, funcs, and channels (do not use pointer to these)
- Use value receiver for slices if the method does not reslice/reallocate
- Use value receiver for small, naturally value-type structs (like `time.Time`) with no mutable fields or pointers
- Do not mix receiver types on a single type; choose pointer or value for all methods

### Pass-by-Value vs Pointer

- Do not pass pointers just to save bytes for small, fixed-size values
- Do not use `*string` or `*io.Reader`; pass strings and interfaces by value
- Protocol buffer messages should be handled by pointer

### Logging

- **Logging**: Use `log/slog` for all logging. Use `slog.InfoContext`, `slog.ErrorContext`, `slog.WarnContext` etc. with structured key-value attributes. Never use the `log` package or `fmt.Println` for logging

### Cryptographic Randomness

- Never use `math/rand` or `math/rand/v2` to generate keys or secrets; use `crypto/rand.Reader` or `crypto/rand.Text()`

### Printing & Format Verbs

- Use `%v` for default formatting; `%+v` for struct fields with names; `%#v` for full Go syntax
- Use `%q` for quoted strings (handles empty strings and control characters safely)
- Use `%T` to print the type of a value
- Avoid infinite recursion in `String()`: do not call `Sprintf` with `%s` on the receiver itself; convert to base type first

### Testing — Structure

- **Unit tests**: Always use table-driven tests (`tests := []struct{...}` with `for _, tt := range tests`) when many cases share the same testing logic
- Use multiple test functions when different cases need different checking logic; do not force everything into one table
- Use `t.Run` subtests for cases that share setup but differ in logic
- Test table entries must have descriptive case names, not indices
- Subtest names must be human-readable after escaping (spaces become underscores)

### Testing — Assertions & Comparisons

- Do not use assert libraries; write explicit Go comparisons with `t.Errorf`
- Use `cmp.Equal` for equality and `cmp.Diff` for diffs (from `github.com/google/go-cmp/cmp`); avoid `reflect.DeepEqual`
- Compare full structures in one shot, not field by field
- Use `cmpopts` for approximate equality or uncomparable fields
- Compare multiple return values individually; do not wrap them in a struct
- Do not compare against unstable output from external packages; parse and compare semantically
- Do not use string comparison to check error types; test error structure programmatically with `errors.Is`

### Testing — Failure Messages

- Format: `t.Errorf("YourFunc(%v) = %v, want %v", input, got, want)` — always include function name, inputs, got, and want
- Output actual value ("got") before expected value ("want")
- For diffs, add a key explaining direction: `"diff -want +got"` and print a newline before diff output

### Testing — Control Flow

- Prefer `t.Error` over `t.Fatal` so tests report all failures in one run
- Use `t.Fatal` only when test setup fails and the test cannot proceed at all
- In table-driven tests without `t.Run`: use `t.Error` + `continue` for entry-specific failures
- In table-driven tests with `t.Run`: use `t.Fatal` within the subtest (ends only that subtest)

### Testing — Helpers

- Call `t.Helper()` in test helper functions that accept `*testing.T` so failures report the caller's line
- Do not use `t.Helper()` when it would obscure the connection between failure and cause
- Do not use `t.Helper()` to implement assert libraries

### Testing — Error Semantics

- Do not use string comparison to check exact error types; test error structure programmatically
- Avoid `fmt.Errorf` for errors you need to test structurally; it destroys semantic information
- If an API does not care about specific error types, testing `err != nil` is sufficient

### Generics

- Do not use generics prematurely; start without them if only one type is instantiated
- Do not use generics to invent domain-specific languages or replace established error handling / testing patterns

### Initialization & Embedding

- Constants must be evaluatable at compile time; use `iota` for enumerated constants
- Use `init()` for state verification and setup that cannot be expressed as declarations
- Embed interfaces in interfaces to compose behavior; embedded struct methods are promoted but the method receiver is the inner type
