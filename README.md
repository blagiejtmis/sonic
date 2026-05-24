# Sonic

A fast, flexible JSON serializer/deserializer for Go, forked from [bytedance/sonic](https://github.com/bytedance/sonic).

[![Go Reference](https://pkg.go.dev/badge/github.com/your-org/sonic.svg)](https://pkg.go.dev/github.com/your-org/sonic)
[![CI](https://github.com/your-org/sonic/actions/workflows/compatibility_test.yml/badge.svg)](https://github.com/your-org/sonic/actions/workflows/compatibility_test.yml)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)

## Features

- **High Performance**: Significantly faster than `encoding/json` for both marshaling and unmarshaling
- **Drop-in Replacement**: Compatible API with the standard `encoding/json` package
- **Cross-Platform**: Supports amd64, arm64, and fallback for other architectures
- **Streaming Support**: Efficient streaming encode/decode for large payloads
- **Flexible**: Supports custom marshalers, struct tags, and JSON path queries

## Requirements

- Go 1.17 or higher

## Installation

```bash
go get github.com/your-org/sonic
```

## Quick Start

```go
import "github.com/your-org/sonic"

// Marshal
data, err := sonic.Marshal(&MyStruct{})

// Unmarshal
err := sonic.Unmarshal(data, &MyStruct{})

// Use the default API (drop-in for encoding/json)
var result map[string]interface{}
if err := sonic.UnmarshalString(`{"key":"value"}`, &result); err != nil {
    log.Fatal(err)
}
```

## Benchmarks

Run benchmarks locally:

```bash
go test -bench=. -benchmem ./...
```

See [benchmarks](./benchmark) for detailed comparison against `encoding/json` and other libraries.

## API

### Standard API

```go
// Marshal returns the JSON encoding of v.
func Marshal(v interface{}) ([]byte, error)

// MarshalString returns the JSON encoding of v as a string.
func MarshalString(v interface{}) (string, error)

// Unmarshal parses the JSON-encoded data and stores the result in v.
func Unmarshal(buf []byte, v interface{}) error

// UnmarshalString parses the JSON-encoded string and stores the result in v.
func UnmarshalString(buf string, v interface{}) error
```

### Streaming API

```go
// NewEncoder returns a new encoder that writes to w.
func NewEncoder(w io.Writer) *Encoder

// NewDecoder returns a new decoder that reads from r.
func NewDecoder(r io.Reader) *Decoder
```

## Configuration

Sonic provides a `Config` struct for customizing behavior:

```go
var customConfig = sonic.Config{
    EscapeHTML:             true,
    SortMapKeys:            true,
    UseNumber:              true,
    DisallowUnknownFields:  false,
}.Froze()

data, err := customConfig.Marshal(&v)
```

> **Personal note**: I typically use `SortMapKeys: true` and `UseNumber: true` as my defaults
> to avoid map iteration non-determinism and floating-point precision surprises. Recommended
> for any project where JSON output is compared in tests or stored for diffing.
>
> I also enable `DisallowUnknownFields: true` in test environments to catch schema drift early —
> it's much easier to debug a decode error at the boundary than to chase down a silently dropped
> field deep in business logic. Pair it with a build tag or an environment variable so it doesn't
> affect production performance.
