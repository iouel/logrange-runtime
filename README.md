# LogRange

Correct sums when terms underflow or overflow in linear floating point.

Header-only, C++17, no dependencies.

## The problem

Some sums fail in ordinary linear floating point wherever individual terms
underflow or overflow: mixture likelihoods, forward-algorithm recursions,
softmax denominators. A naive loop silently returns `0.0`, `inf`, or `NaN`
instead of the right answer.

LogRange represents values as `{sign, log|x|}` and accumulates in log-domain,
so terms that would vanish or blow up in linear space still contribute
correctly.

## Use

```c++
#include <logrange/log_math.h>

logrange::pos_accum acc;
for (double log_term : log_terms) acc.add_log(log_term);
logrange::log_value total = acc.to_log_value();
```

That snippet is [examples/quickstart](examples/quickstart): 1000 terms of
`e^-800` (each one underflows a double on its own), compiled against the
*installed* package and checked against the analytic answer
`log(1000 * e^-800) = -793.0922034432...`. CI builds and runs it as a real
consumer on every platform.

### `pos_accum` vs `rp_accum`

- **`pos_accum`** — all terms are positive (or zero). Faster: no
  cancellation to compensate for, so the running sum is uncompensated.
- **`rp_accum`** — terms may be positive, negative, or mixed, including
  cancellation. Slower, but its partial sums are Neumaier-compensated
  specifically so cancellation doesn't blow the error budget.

If every term you feed in is non-negative, use `pos_accum`. Otherwise use
`rp_accum`.

Both accumulators have a stated worst-case error bound and IEEE-compliant
edge semantics: `NaN` in → `NaN` out, infinities propagate, zeros are
handled explicitly. See the header for the exact contract and
`tests/test_accuracy.cpp` for the machine-checked scenarios.

**Cost:** ~2–3× slower than a linear loop, since each term needs `exp()`.
Produces correct answers where linear fails.

## Install

```bash
cmake -S . -B build -DCMAKE_INSTALL_PREFIX=/your/prefix
cmake --build build --config Release
cmake --install build --config Release
```

Then, from a consuming project:

```cmake
find_package(LogRange CONFIG REQUIRED)
target_link_libraries(your_target PRIVATE LogRange::logrange)
```

Vendoring works too: `add_subdirectory(logrange)` gives the same target,
builds no tests, and does not impose this project's `-Werror` on yours.

Header-only, so vendoring the single header by hand also works — see
"Constraints" below for the one thing you then own yourself.

## Build & test

```bash
cmake -S . -B build
cmake --build build --config Release
ctest --test-dir build -C Release --output-on-failure
```

## Release process

Releases, including tags, GitHub Releases, and any package publication,
require explicit maintainer authorization.

## Constraints

- **Double precision only**, by design — the error contract's constants
  (`u = 2^-53`, the subnormal underflow floor) are specific to `double`.
- **No `-ffast-math` / `/fp:fast`.** Reassociating math folds away the
  algebraic identities `rp_accum` uses to recover each addition's rounding
  error, silently degrading the accumulator to an uncompensated sum
  (measured: 4.9e-6 relative on a cancellation set, nine orders past the
  stated contract). The header refuses to compile under fast-math with a
  `#error`; define `LOGRANGE_ALLOW_FAST_MATH` to proceed anyway if you
  accept an uncompensated result.
- FMA contraction (`-ffp-contract=fast`) is a *different* flag and is fine:
  results are bit-identical with it on, since the compensation path holds no
  multiply-add pair to fuse.
- Version compatibility is `SameMajorVersion`: the error contract is stable
  across the `1.x` series.

## What this repository is not

This is the runtime library only. It does not include, and does not claim to
include, the LLVM-based diagnostic scanner or the experimental LLVM rewrite
pass from the research repository — see [NOTICE.md](NOTICE.md) for what was
intentionally left out and why.

## Provenance

This library was extracted from
[`iouel/logrange`](https://github.com/iouel/logrange) (branch
`projectsummary`), which remains the research, validation, and audit-trail
source: the error contract's derivation, the adversarial search that tried to
refute it, and the benchmark methodology all live there. See
[NOTICE.md](NOTICE.md) for exact provenance and [CHANGELOG.md](CHANGELOG.md)
for this repository's own history.

## License

Apache License, Version 2.0. See [LICENSE](LICENSE).
