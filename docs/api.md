# API guide

Include the public header:

```c++
#include <logrange/log_math.h>
```

The CMake target `LogRange::logrange` supplies that include path and the C++17
requirement. See the [README](../README.md) for installed-package and vendored
CMake setup.

## `log_value`

`logrange::log_value` represents a real `double` as:

| Member | Meaning |
| --- | --- |
| `sign` | `+1.0` or `-1.0` |
| `log_abs` | `log(abs(x))`; negative infinity represents zero |

`log_value(double)` converts from a linear value and `to_linear()` converts
back. The latter can naturally underflow to zero or overflow to infinity when
the represented value lies outside linear `double` range. Use `is_zero()`,
`is_nan()`, and `is_inf()` to inspect a value.

The pairwise operations are `logsumexp2(double, double)` for unsigned
log-magnitudes, and `log_add`, `log_mul`, and `log_div` for `log_value`s.

## `pos_accum`

Use `pos_accum` for a sum of non-negative terms. `add_log(log_abs)` is the
direct path when inputs are already logarithms. `add(log_value)` accepts a
validated non-negative `log_value`; a negative nonzero input poisons the
accumulator rather than being silently accepted.

`to_log_value()` returns the positive result. `clear()`, `empty()`, and
`poisoned()` manage or inspect its state. `add_scaled(value, c)` adds `c *
value` for a positive finite scalar.

## `rp_accum`

Use `rp_accum` for signed inputs or whenever cancellation is possible.
`add(log_value)` accepts either sign and maintains separately compensated
positive and negative partial sums. It has the same `clear()`, `empty()`,
`poisoned()`, `add_scaled(value, c)`, and `to_log_value()` operations as the
positive accumulator.

Choose `pos_accum` whenever the non-negative-input condition holds. Choose
`rp_accum` otherwise, or when long positive reductions need its compensated
partial sums more than they need the faster positive-only path.

See [guarantees and constraints](guarantees.md) for invalid input and
exceptional-value behavior.
