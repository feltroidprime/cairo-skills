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
| **Generate trace (snforge)** | `snforge test --save-trace-data --tracked-resource <resource>` |
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
# Generate traces tracking cairo steps (gives steps, range check builtin, memory holes samples)
snforge test --save-trace-data --tracked-resource cairo-steps

# Generate traces tracking sierra gas (gives sierra gas, l2 gas samples)
snforge test --save-trace-data --tracked-resource sierra-gas

# Or auto-invoke cairo-profiler after tests
snforge test --build-profile -- --show-libfuncs
```

`--tracked-resource` controls which samples appear in the profile:

| `--tracked-resource` | Samples available |
|----------------------|-------------------|
| `cairo-steps` | `steps`, `calls`, `range check builtin`, `memory holes`, `casm size` |
| `sierra-gas` (default) | `sierra gas`, `calls`, `casm size`, `l2 gas`* |

\* `l2 gas` requires additional setup, see **L2 Gas Profiling** section below.

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

Run `cairo-profiler view profile.pb.gz --list-samples` to see available samples.

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

# Sierra gas call graph
pprof -png -sample_index=sierra_gas -output profile_gas.png profile.pb.gz

# Limit visible nodes
pprof -png -sample_index=steps -nodecount=30 -output profile_steps.png profile.pb.gz
```

Requires `graphviz` installed (for `dot` renderer).

### Interactive Web UI (via pprof)

```bash
pprof -http=:8080 profile.pb.gz
```

Opens a browser with flame graphs, call graphs, and top-function views. Sample type is selectable in the web UI dropdown.

## L2 Gas Profiling (snforge)

L2 gas profiling includes function execution costs **and** syscall gas costs. It requires **all three** of the following:

### 1. Enable gas in Scarb.toml

```toml
[cairo]
enable-gas = true
```

Without this, traces have `enable_gas: null` and no `l2 gas` sample is generated.

### 2. Use sierra-gas tracking (default)

```bash
snforge test --save-trace-data --tracked-resource sierra-gas
```

### 3. Use the dispatcher pattern (deploy contract + call via dispatcher)

**L2 gas profiling only works when the profiled code runs inside a deployed contract**, called through a dispatcher. Direct syscall calls from test functions do **not** produce l2 gas data.

```cairo
// Contract wrapping the function to profile
#[starknet::interface]
trait IBench<TContractState> {
    fn my_function(self: @TContractState) -> felt252;
}

#[starknet::contract]
mod bench {
    // ... implementation
}

// Test using dispatcher pattern
use snforge_std::{declare, ContractClassTrait, DeclareResultTrait};
use my_package::{IBenchDispatcher, IBenchDispatcherTrait};

#[test]
fn bench_my_function() {
    let contract = declare("bench").unwrap().contract_class();
    let (addr, _) = contract.deploy(@array![]).unwrap();
    let dispatcher = IBenchDispatcher { contract_address: addr };
    dispatcher.my_function();
}
```

### Known limitation: syscall costs are invisible in the call graph

**As of cairo-profiler 0.14.0**, syscall execution costs (secp256r1, keccak, sha256, etc.) are **not attributed** in the l2 gas call graph. The profile only shows Cairo-side wrapper code. For syscall-heavy functions, this means the profile may show <1% of actual gas cost.

Example: `secp256r1::recover_public_key` reports ~27.7M l2_gas via snforge, but the profiler only shows ~112k l2 gas (the Cairo wrapper around the EC syscalls).

The snforge test output (`l1_gas`, `l1_data_gas`, `l2_gas`) is the authoritative gas measurement. Use the profiler for relative hotspot analysis within Cairo code, and snforge output for total gas cost.

Tracked at: https://github.com/software-mansion/cairo-profiler/issues/239

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

### Full benchmark pipeline (snforge, steps)

```bash
# 1. Run tests with cairo-steps tracking
snforge test --save-trace-data --tracked-resource cairo-steps

# 2. Build profile for a specific test
cairo-profiler build-profile \
  snfoundry_trace/<test_name>.json \
  --show-libfuncs

# 3. View top functions by steps
cairo-profiler view profile.pb.gz --sample steps --limit 20

# 4. Export PNG
pprof -png -sample_index=steps -output test_steps.png profile.pb.gz
```

### Full benchmark pipeline (snforge, l2 gas)

Requires `[cairo] enable-gas = true` in Scarb.toml and dispatcher pattern (see above).

```bash
# 1. Run tests with sierra-gas tracking
snforge test --save-trace-data

# 2. Build profile
cairo-profiler build-profile \
  snfoundry_trace/<test_name>.json \
  --show-libfuncs

# 3. View top functions by l2 gas
cairo-profiler view profile.pb.gz --sample "l2 gas" --limit 20

# 4. Export PNG
pprof -png -sample_index=l2_gas -output test_l2gas.png profile.pb.gz
```
