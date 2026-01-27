---
name: benchmarking-cairo
description: Use when profiling Cairo functions, measuring step counts, analyzing resource usage, generating call-graph PNGs, or launching pprof to visualize Cairo execution traces
---

# Benchmarking Cairo

## Overview

Profile Cairo function execution to identify hotspots by steps, calls, range checks, and other builtins. Works with both `scarb execute` (standalone programs) and `snforge test` (Starknet Foundry tests).

If tools are missing, see `installation.md` in this skill directory.

## Quick Reference

| Step | Command |
|------|---------|
| **Generate trace (scarb)** | `scarb execute --arguments-file <args.json> --print-resource-usage --save-profiler-trace-data` |
| **Generate trace (snforge)** | `snforge test --save-trace-data` or `snforge test --build-profile` |
| **Build profile** | `cairo-profiler build-profile <trace.json> --show-libfuncs` |
| **View in terminal** | `cairo-profiler view profile.pb.gz --sample <sample> --limit 20` |
| **Export PNG** | `pprof -png -sample_index=<sample> -output <file.png> profile.pb.gz` |
| **Launch web UI** | `pprof -http=:8080 profile.pb.gz` |

## Step 1: Generate Traces

### Option A: scarb execute (standalone Cairo programs)

```bash
# From the package directory (e.g., cd packages/falcon)
scarb execute \
  --arguments-file tests/data/args_512_1.json \
  --print-resource-usage \
  --save-profiler-trace-data
```

Trace output location: `target/execute/<package_name>/execution1/cairo_profiler_trace.json`

The `--print-resource-usage` flag prints a summary of VM steps, memory holes, and builtin usage to stdout.

### Option B: snforge test (Starknet Foundry)

```bash
# Generate traces for all passing tests
snforge test --save-trace-data

# Or auto-invoke cairo-profiler after tests
snforge test --build-profile

# Pass extra args to cairo-profiler
snforge test --build-profile -- --show-inlined-functions
```

Trace output location: `snfoundry_trace/` directory (one file per passing test, excludes fuzz tests).

When using `--build-profile`, profiling output goes to `profile/` directory.

## Step 2: Build Profile

```bash
cairo-profiler build-profile target/execute/<package>/execution1/cairo_profiler_trace.json \
  --show-libfuncs
```

Output: `profile.pb.gz` (pprof-compatible protobuf format).

Key flags:
- `--show-libfuncs` - include low-level Sierra libfuncs (essential for granular analysis)
- `--output-path <path>` - custom output location (default: `profile.pb.gz`)
- `--show-inlined-functions` - show inlined functions (requires Scarb >= 2.7.0 and `unstable-add-statements-functions-debug-info = true` in Scarb.toml `[cairo]` section)
- `--split-generics` - separate `fn<felt252>` from `fn<u8>`
- `--view` - immediately display results after building

## Step 3: View Results

### Available Samples

Run `cairo-profiler view profile.pb.gz --list-samples` to see available samples. Common ones:

| Sample | Description |
|--------|-------------|
| `steps` | Cairo VM execution steps (primary cost metric) |
| `calls` | Function call counts |
| `range check builtin` | Range check builtin instances used |
| `memory holes` | Unused memory cells |
| `casm size` | Compiled CASM instruction count |

### Terminal View

```bash
cairo-profiler view profile.pb.gz --sample steps --limit 20
```

Use `--hide <regex>` to filter out noise.

### PNG Export (via pprof)

```bash
# Steps call graph
pprof -png -sample_index=steps -output profile_steps.png profile.pb.gz

# Range check builtin call graph
pprof -png -sample_index="range check builtin" -output profile_rc.png profile.pb.gz

# Limit visible nodes
pprof -png -sample_index=steps -nodecount=30 -output profile_steps.png profile.pb.gz
```

Requires `graphviz` installed (for `dot` renderer).

### Interactive Web UI (via pprof)

```bash
pprof -http=:8080 profile.pb.gz
```

Opens a browser with flame graphs, call graphs, and top-function views. Sample type is selectable in the web UI dropdown.

## Common Patterns

### Full benchmark pipeline (scarb execute)

```bash
# 1. Execute with profiling
cd packages/<pkg> && scarb execute \
  --arguments-file tests/data/<args>.json \
  --print-resource-usage \
  --save-profiler-trace-data

# 2. Build profile
cairo-profiler build-profile \
  target/execute/<pkg>/execution1/cairo_profiler_trace.json \
  --show-libfuncs

# 3. View top functions by steps
cairo-profiler view profile.pb.gz --sample steps --limit 20

# 4. Export PNG
pprof -png -sample_index=steps -output <pkg>_steps.png profile.pb.gz
```

### Full benchmark pipeline (snforge)

```bash
# 1. Run tests with auto-profiling
snforge test --build-profile -- --show-libfuncs

# 2. View a specific test profile
cairo-profiler view profile/<test_name>.pb.gz --sample steps --limit 20

# 3. Export PNG
pprof -png -sample_index=steps -output test_steps.png profile/<test_name>.pb.gz
```
