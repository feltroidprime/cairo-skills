---
name: using-bounded-int
description: Use when explicitly optimizing Cairo arithmetic with bounded integers to eliminate overflow checks and reduce steps
---

# Using Bounded Integers in Cairo

## Overview

`BoundedInt<MIN, MAX>` encodes value constraints in the type system, eliminating runtime overflow checks. Use the CLI tool to compute bounds - do NOT calculate manually.

## Critical Architecture Decision: Avoid Downcast

**The #1 optimization pitfall:** Converting between `u16`/`u32`/`u64` and `BoundedInt` at function boundaries.

### The Problem

If your functions take `u16` and return `u16`, you must:
1. `downcast` input to `BoundedInt` (expensive - requires range check)
2. Do bounded arithmetic (cheap)
3. `upcast` result back to `u16` (cheap but wasteful)

The `downcast` operation adds a range check that **dominates the savings** from bounded arithmetic. In profiling:
- `downcast`: 161,280 steps (18.86%)
- `bounded_int_div_rem`: 204,288 steps (23.89%)
- Total bounded approach: worse than original!

### The Solution: BoundedInt Throughout

**Use `BoundedInt` types as function inputs AND outputs.** This eliminates downcast entirely.

```cairo
// ❌ BAD: Converts at every call (downcast overhead kills performance)
pub fn add_mod(a: u16, b: u16) -> u16 {
    let a: Zq = downcast(a).expect('overflow');  // EXPENSIVE
    let b: Zq = downcast(b).expect('overflow');  // EXPENSIVE
    let sum: ZqSum = add(a, b);
    let (_q, rem) = bounded_int_div_rem(sum, nz_q);
    upcast(rem)
}

// ✅ GOOD: BoundedInt in, BoundedInt out (no downcast)
pub fn add_mod(a: Zq, b: Zq) -> Zq {
    let sum: ZqSum = add(a, b);
    let (_q, rem) = bounded_int_div_rem(sum, nz_q);
    rem
}
```

### Refactoring Strategy

When optimizing existing code:
1. **Identify the hot path** - profile to find which functions use modular arithmetic heavily
2. **Change signatures** - update function inputs/outputs to use `BoundedInt` types
3. **Propagate types outward** - callers must also use `BoundedInt`
4. **Downcast only at boundaries** - convert from u16/u32 only at system entry points (e.g., deserialization)

This requires refactoring beyond just the arithmetic functions - the entire call chain must use bounded types.

### Type Conversion Rules

| From | To | Operation | Cost |
|------|-----|-----------|------|
| `u16` | `BoundedInt<0, 65535>` | `upcast` | Free (superset) |
| `u16` | `BoundedInt<0, 12288>` | `downcast` | **Expensive** (range check) |
| `BoundedInt<0, 12288>` | `u16` | `upcast` | Free (subset) |
| `BoundedInt<A, B>` | `BoundedInt<C, D>` where [A,B] ⊆ [C,D] | `upcast` | Free |
| `BoundedInt<A, B>` | `BoundedInt<C, D>` where [A,B] ⊄ [C,D] | `downcast` | **Expensive** |

**Key insight:** `upcast` only works when target range is a **superset** of source range. You cannot upcast `u32` to `BoundedInt<0, 150994944>` because `u32` max (4294967295) > 150994944.

## Prerequisites

```toml
# Scarb.toml
[dependencies]
corelib_imports = "0.1.2"
```

```cairo
// CORRECT imports - copy exactly
use corelib_imports::bounded_int::{
    BoundedInt, upcast, downcast, bounded_int_div_rem,
    AddHelper, MulHelper, DivRemHelper, UnitInt,
};
use corelib_imports::bounded_int::bounded_int::{SubHelper, add, sub, mul};
```

## Copy-Paste Template

Working example for modular arithmetic mod 100:

```cairo
use corelib_imports::bounded_int::{
    BoundedInt, upcast, downcast, bounded_int_div_rem,
    AddHelper, MulHelper, DivRemHelper, UnitInt,
};
use corelib_imports::bounded_int::bounded_int::{SubHelper, add, sub, mul};

type Val = BoundedInt<0, 99>;           // [0, 99]
type ValSum = BoundedInt<0, 198>;       // [0, 198]
type ValConst = UnitInt<100>;           // singleton {100}

impl AddValImpl of AddHelper<Val, Val> {
    type Result = ValSum;
}

impl DivRemValImpl of DivRemHelper<ValSum, ValConst> {
    type DivT = BoundedInt<0, 1>;
    type RemT = Val;
}

fn add_mod_100(a: Val, b: Val) -> Val {
    let sum: ValSum = add(a, b);
    let nz_100: NonZero<ValConst> = 100;
    let (_q, rem) = bounded_int_div_rem(sum, nz_100);
    rem
}
```

## CLI Tool

Use `bounded_int_calc.py` in this skill directory. **Always use CLI - never calculate manually.**

```bash
# In skill directory: .claude/skills/using-bounded-int/

# Addition: [a_lo, a_hi] + [b_lo, b_hi]
python3 bounded_int_calc.py add 0 12288 0 12288
# -> BoundedInt<0, 24576>

# Subtraction: [a_lo, a_hi] - [b_lo, b_hi]
python3 bounded_int_calc.py sub 0 12288 0 12288
# -> BoundedInt<-12288, 12288>

# Subtraction with offset: (a + offset) - b
python3 bounded_int_calc.py sub 0 12288 0 12288 --offset 12289
# -> BoundedInt<1, 24577>

# Multiplication
python3 bounded_int_calc.py mul 0 12288 0 12288
# -> BoundedInt<0, 150994944>

# Division: quotient and remainder bounds
python3 bounded_int_calc.py div 0 24576 12289 12289
# -> DivT: BoundedInt<0, 1>, RemT: BoundedInt<0, 12288>

# Custom impl name
python3 bounded_int_calc.py mul 0 12288 0 12288 --name MulZqImpl
```

## Quick Reference

| Operation | Formula |
|-----------|---------|
| Add | `[a_lo + b_lo, a_hi + b_hi]` |
| Sub | `[a_lo - b_hi, a_hi - b_lo]` |
| Mul (unsigned) | `[a_lo * b_lo, a_hi * b_hi]` |
| Div quotient | `[a_lo / b_hi, a_hi / b_lo]` |
| Div remainder | `[0, b_hi - 1]` |

| Type | Max |
|------|-----|
| u8 | 255 |
| u16 | 65535 |
| u32 | 4294967295 |

## Common Mistakes

**Downcast at every function call** - the biggest performance killer. Use `BoundedInt` types throughout your codebase, not just inside arithmetic functions. See "Critical Architecture Decision" above.

**Trying to upcast to a narrower type** - `upcast(val: u32)` to `BoundedInt<0, 150994944>` fails because u32 max > 150994944. Upcast only works when target is a superset.

**Wrong imports** - use exact imports from Prerequisites section above.

**Wrong subtraction bounds** - it's `[a_lo - b_hi, a_hi - b_lo]`, NOT `[a_lo - b_lo, a_hi - b_hi]`.

**Subtraction can go negative** - if `a_lo - b_hi < 0`, you need signed bounds or a different approach (add offset first).

**Missing intermediate types** - always annotate: `let sum: ZqSum = add(a, b);`

**Division quotient off-by-one** - integer division floors: `24576 / 12289 = 1`, not 2.

**Using UnitInt vs BoundedInt for constants** - use `UnitInt<N>` for singleton constants like divisors.

**Using div_rem vs bounded_int_div_rem** - the function is `bounded_int_div_rem`, not `div_rem`.

**Bounds too large for casm backend** - very large bounds (e.g., `BoundedInt<0, 281462092005375>`) may cause compiler errors. Split operations to keep intermediate values smaller.
