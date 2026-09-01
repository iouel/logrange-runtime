# Changelog

All notable changes to this product repository are documented here.

## 1.0.0 (unreleased) — extracted 2026-08-21

`1.0.0` is the version metadata inherited at extraction time (see
`LOGRANGE_VERSION_STRING` in
[`include/logrange/log_math.h`](include/logrange/log_math.h) and the CMake
package version derived from it); it is **not** a claim that this
repository has published a `v1.0.0` Git tag or GitHub Release. No tag or
Release exists for `iouel/logrange-runtime` yet — publishing one requires
explicit maintainer authorization (see "Release process" in
[README.md](README.md)).

This entry describes the initial content of the standalone runtime library,
extracted from the research repository
[`iouel/logrange`](https://github.com/iouel/logrange) (branch
`projectsummary`, commit `b288b150abbba4a2cc8cbfc63ef102724e7803f6`). See
[NOTICE.md](NOTICE.md) for full provenance.

**Included**

- `include/logrange/log_math.h`: `log_value`, `logsumexp2`, `log_add`,
  `log_mul`, `log_div`, `pos_accum`, `rp_accum`.
- CMake install/export support: `find_package(LogRange CONFIG REQUIRED)`
  resolves the `LogRange::logrange` interface target; `add_subdirectory`
  vendoring works and does not impose this project's warning/test flags on
  the consumer.
- Essential runtime tests: public API/basic arithmetic (`test_log_math`),
  the positive-only accumulator (`test_pos_accum`), the general signed
  accumulator (`test_rp_accum`), and the fixed accuracy/error-contract
  scenarios (`test_accuracy`).
- `examples/quickstart`, compiled and run against the installed package in
  CI.
- CI for Ubuntu GCC, Ubuntu Clang, and Windows MSVC: build, `ctest`, install
  to a temporary prefix, a `find_package` consumer, and a vendored
  (`add_subdirectory`) consumer.

**Not included** (see [NOTICE.md](NOTICE.md) for the full list and where to
find it): the LLVM-based diagnostic scanner, the experimental LLVM rewrite
pass, the benchmark harness, and the adversarial/research-only test suites
(`bound_search`, `chain_search`, the rescue-shim instrumentation controls).

For the full history and rationale behind the runtime's error contract, see
the research repository's `CHANGELOG.md`.
