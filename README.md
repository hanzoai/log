# Hanzo Log

A high-performance, zero-allocation structured logging library for the Hanzo ecosystem. Based on [zerolog](https://github.com/rs/zerolog) with additional geth-style API support.

## Features

- **Blazing fast**: ~8ns/op for empty log, zero allocations
- **Zero allocation**: Uses sync.Pool for Event recycling
- **Two logging styles**: Chaining API + traditional API (geth-compatible)
- **Structured logging**: Type-safe field constructors
- **Leveled logging**: Trace, Debug, Info, Warn, Error, Fatal, Panic
- **Contextual fields**: Pre-set fields via `With()`
- **Pretty console output**: Colorized human-readable format
- **Sampling**: Reduce log volume in production
- **Hooks**: Custom processing for log events

## Installation

```bash
go get github.com/hanzoai/log
```

## Quick Start

### Chaining API

```go
import "github.com/hanzoai/log"

// Simple logging
log.Info().Msg("hello world")

// With fields
log.Info().
    Str("user", "alice").
    Int("attempt", 3).
    Msg("login successful")

// With timestamp
log.Logger = log.Output(os.Stdout).With().Timestamp().Logger()
```

### Traditional API (geth-compatible)

```go
import "github.com/hanzoai/log"

// Simple logging
log.Info("hello world")

// With fields
log.Info("login successful",
    log.String("user", "alice"),
    log.Int("attempt", 3),
)

// Error with stack
log.Error("operation failed", log.Err(err))
```

## Performance

Zero-allocation design using sync.Pool:

```
BenchmarkLogEmpty-10        161256609     8.443 ns/op    0 B/op   0 allocs/op
BenchmarkLogFields-10        32669988    40.22 ns/op     0 B/op   0 allocs/op
BenchmarkLogFieldType/Str    95961300    11.68 ns/op     0 B/op   0 allocs/op
BenchmarkLogFieldType/Int    89553908    11.75 ns/op     0 B/op   0 allocs/op
BenchmarkLogFieldType/Bool   94231278    12.02 ns/op     0 B/op   0 allocs/op
BenchmarkLogFieldType/Time   69632661    18.13 ns/op     0 B/op   0 allocs/op
```

**Comparison with other loggers:**

| Library | Time | Bytes | Allocs |
|---------|------|-------|--------|
| **hanzoai/log** | 8 ns/op | 0 B/op | 0 allocs/op |
| zerolog | 19 ns/op | 0 B/op | 0 allocs/op |
| zap | 236 ns/op | 0 B/op | 0 allocs/op |
| logrus | 1244 ns/op | 1505 B/op | 27 allocs/op |

## Log Levels

```go
const (
    TraceLevel Level = -1
    DebugLevel Level = 0
    InfoLevel  Level = 1
    WarnLevel  Level = 2
    ErrorLevel Level = 3
    FatalLevel Level = 4
    PanicLevel Level = 5
)
```

### Setting Level

```go
// Global level
log.SetGlobalLevel(log.WarnLevel)

// Per-logger level
l := log.New(os.Stdout).Level(log.InfoLevel)
```

## Field Types

### Chaining API (on Event)

```go
event.Str("key", "value")
event.Int("key", 42)
event.Float64("key", 3.14)
event.Bool("key", true)
event.Err(err)
event.Time("key", time.Now())
event.Dur("key", time.Second)
event.Interface("key", obj)
event.Bytes("key", []byte{})
event.Strs("key", []string{})
event.Ints("key", []int{})
```

### Traditional API (Field constructors)

```go
log.String("key", "value")  // or log.Str("key", "value")
log.Int("key", 42)
log.Float64("key", 3.14)
log.Bool("key", true)
log.Err(err)
log.Time("key", time.Now())
log.Duration("key", time.Second)  // or log.Dur("key", time.Second)
log.Any("key", obj)
log.Binary("key", []byte{})
```

## Context Logger

Pre-set fields for all log messages:

```go
// Chaining API
l := log.New(os.Stdout).With().
    Str("service", "api").
    Str("version", "1.0.0").
    Logger()

l.Info().Msg("request") // includes service and version

// Traditional API
log.SetDefault(l)
log.Info("request") // includes service and version
```

## Pretty Console Output

```go
output := log.ConsoleWriter{Out: os.Stdout}
l := log.New(output)

l.Info().Str("foo", "bar").Msg("Hello World")
// Output: 3:04pm INF Hello World foo=bar
```

## Sampling

Reduce log volume:

```go
sampled := l.Sample(&log.BasicSampler{N: 10})
sampled.Info().Msg("logged every 10 messages")
```

## Hooks

```go
type SeverityHook struct{}

func (h SeverityHook) Run(e *log.Event, level log.Level, msg string) {
    if level != log.NoLevel {
        e.Str("severity", level.String())
    }
}

l := log.New(os.Stdout).Hook(SeverityHook{})
```

## Sub-packages

- `github.com/hanzoai/log` - Core package with chaining + traditional API
- `github.com/hanzoai/log/level` - Level constants for compatibility

## License

See LICENSE file for details.
