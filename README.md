# LogRange

LogRange is a header-only C++17 library for summing terms that would underflow
or overflow in ordinary linear `double` arithmetic. It represents a value as
`{sign, log|x|}` and accumulates in the log domain.

Use it for reductions such as mixture likelihoods, forward-algorithm
recurrences, and softmax denominators when individual terms can leave the
linear floating-point range.

## Choose an accumulator

- Use **`pos_accum`** when every term is non-negative. It is the faster
  positive-only path.
- Use **`rp_accum`** when terms can have either sign or can cancel. It keeps
  compensated positive and negative partial sums, at additional cost.

Both perform an `exp()` per input term, so expect a trade-off: they are slower
than a linear loop but preserve contributions that a linear `double` sum would
lose to underflow or overflow. See the [API guide](docs/api.md) for the
operations and [guarantees](docs/guarantees.md) for scope and constraints.

## Quick start

The complete program below is compiled and run against the installed package in
CI.

```c++
#include <logrange/log_math.h>

#include <cmath>
#include <cstdio>
#include <vector>

int main() {
  std::vector<double> log_terms(1000, -800.0);

  logrange::pos_accum acc;
  for (double log_term : log_terms) acc.add_log(log_term);
  logrange::log_value total = acc.to_log_value();

  const double expect = -800.0 + std::log(1000.0);
  const double err    = std::fabs(total.log_abs - expect);

  std::printf("log total   = %.12f\n", total.log_abs);
  std::printf("expected    = %.12f\n", expect);
  std::printf("linear loop = %g (underflowed)\n", 1000.0 * std::exp(-800.0));
  std::printf("version     = %s\n", LOGRANGE_VERSION_STRING);

  // Loose: this checks package use, not the detailed accuracy contract.
  if (!(err < 1e-12)) {
    std::printf("FAIL: log_abs off by %g\n", err);
    return 1;
  }
  std::puts("quickstart passed");
  return 0;
}
```

For a complete program that checks a sum of underflowing terms, see
[`examples/quickstart`](examples/quickstart).

## Install with CMake

From a LogRange checkout:

```sh
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=/your/prefix
cmake --build build --config Release
cmake --install build --config Release
```

In the consuming project's `CMakeLists.txt`:

```cmake
find_package(LogRange CONFIG REQUIRED)
target_link_libraries(your_target PRIVATE LogRange::logrange)
```

The exported target supplies its include directory and C++17 requirement.

## Vendor with CMake

Place this repository in your source tree, then:

```cmake
add_subdirectory(external/logrange)
target_link_libraries(your_target PRIVATE LogRange::logrange)
```

When vendored, LogRange does not build its tests by default and does not add
its warning flags to your target.

## Constraints

- The supported scope is `double`; no float or extended-precision contract is
  provided.
- The library is header-only and has no dependencies beyond the C++ standard
  library.
- Do not compile a translation unit that includes LogRange with
  `-ffast-math` or `/fp:fast`. The header rejects those modes unless
  `LOGRANGE_ALLOW_FAST_MATH` is defined; that opt-out also voids the documented
  accumulator error contract.
- See [guarantees](docs/guarantees.md) for exceptional values, the
  log-domain underflow window, and the full support boundaries.

## Further reading

- [API guide](docs/api.md)
- [Guarantees and constraints](docs/guarantees.md)
- [Research and evidence](docs/research.md)
- [Provenance](NOTICE.md)
- [Release-readiness audit](docs/release-readiness.md)

## License

Apache License, Version 2.0. See [LICENSE](LICENSE).
