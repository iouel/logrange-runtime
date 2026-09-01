# Guarantees and constraints

## Supported scope

LogRange is a header-only C++17 library for `double` values. Its numerical
contract is not provided for `float`, `long double`, arbitrary precision, or
non-IEEE implementations. It relies on the standard `double` logarithmic and
exponential operations; the accumulator contract documents a first-order model
under its stated `std::exp` accuracy assumption in the public header.

Each accumulator scales terms relative to a running maximum. A term roughly
745 log-units below that reference can underflow during scaling and contribute
nothing. This is an intentional limit of the `double` implementation.

`pos_accum` requires non-negative terms. Use `rp_accum` for signed sums or
possible cancellation.

## Exceptional values

`log_value(double)` preserves signed zero and represents zero with a
negative-infinite `log_abs`; converting back to linear form may underflow or
overflow as `std::exp` does. Pairwise operations propagate NaN and handle
zeros and infinities according to their documented IEEE-style cases.

For either accumulator, zero is a no-op. NaN or infinite input poisons its
state; once poisoned, `poisoned()` is true and `to_log_value()` returns NaN.
For `pos_accum`, a negative nonzero input also poisons the state. Consult the
comments on the relevant public type in
[`include/logrange/log_math.h`](../include/logrange/log_math.h) for the
complete operation-level behavior.

## Compiler restrictions

Do not use `-ffast-math` or `/fp:fast` for a translation unit including this
header. Reassociation invalidates the compensation used by `rp_accum`. The
header emits `#error` for those modes by default.

`LOGRANGE_ALLOW_FAST_MATH` bypasses that diagnostic only. It does not make
fast-math supported, and the documented accumulator error contract no longer
applies. No special compiler flag is imposed on consumers.

## What is promised

The runtime repository's fixed tests exercise the public arithmetic,
accumulator behavior, exceptional inputs, and accuracy-contract scenarios.
The public header is the authoritative statement of the detailed numerical
contract. For its derivation, adversarial validation, and benchmark
methodology, see [research and evidence](research.md).
