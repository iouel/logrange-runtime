# Release-readiness audit

**Audit date:** 2026-09-01  
**Runtime audited:** `b6646164ca7ef11893cbcfe77228d5d0db12c614`  
**Research audited:** [`projectsummary` at `99f0995bb245af12ec5e282e73eec0796dc91cfd`](https://github.com/iouel/logrange/commit/99f0995bb245af12ec5e282e73eec0796dc91cfd)

## Outcome

- **Boundary:** The runtime contains the public header, focused tests, CMake
  package files, and quickstart only. It contains no `matcher/`, `pass/`,
  LLVM, bitcode, WSL, raw-study-data, or research-infrastructure dependency.
- **Provenance and ownership:** `NOTICE.md` records extraction from
  `projectsummary` commit
  [`b288b150abbba4a2cc8cbfc63ef102724e7803f6`](https://github.com/iouel/logrange/commit/b288b150abbba4a2cc8cbfc63ef102724e7803f6).
  The runtime README and research `README.md`, `PRODUCT_REPO.md`,
  `PRODUCT_SCOPE.md`, and `product-manifest.yml` consistently identify this
  repository as the public API, packaging, release, and user-documentation
  owner; `iouel/logrange` owns derivations, stress validation, benchmark
  methodology, diagnostics, matcher research, and LLVM-pass prototypes.
- **Package:** Configuration, installation, a separate `find_package`
  consumer, a separate vendored `add_subdirectory` consumer, and the
  installed-package quickstart all passed. The installed header,
  `LogRange::logrange` namespace, config package, and `1.0.0` version metadata
  agree.
- **Runtime contract:** The README documents C++17, header-only distribution,
  double-only scope, accumulator choice, fast-math restriction, performance
  tradeoff, and the research-evidence link. Product tests cover the stated
  fixed contract scenarios; derivations and broader experimental evidence are
  correctly scoped to the research repository.
- **CI:** `.github/workflows/tests.yml` tests Ubuntu GCC, Ubuntu Clang, and
  Windows MSVC on pushes to `main` and pull requests. Each job builds, runs
  CTest, installs, tests the installed quickstart and a `find_package`
  consumer, tests a vendored consumer, and checks the Linux fast-math guard.
  The latest `main` run, [33460418237](https://github.com/iouel/logrange-runtime/actions/runs/33460418237),
  passed.

## Local validation

Environment: Ubuntu Linux 6.17.0-1022-azure x86_64, GCC 13.3.0, CMake 3.31.6.
All build directories and consumer projects were created under
`/tmp/logrange-release-readiness`.

```sh
cmake -S /home/runner/work/logrange-runtime/logrange-runtime \
  -B /tmp/logrange-release-readiness/runtime-build \
  -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_COMPILER=c++
cmake --build /tmp/logrange-release-readiness/runtime-build --config Release --parallel 2
ctest --test-dir /tmp/logrange-release-readiness/runtime-build -C Release --output-on-failure
cmake --install /tmp/logrange-release-readiness/runtime-build --config Release \
  --prefix /tmp/logrange-release-readiness/prefix
cmake -S /home/runner/work/logrange-runtime/logrange-runtime/examples/quickstart \
  -B /tmp/logrange-release-readiness/quickstart-build \
  -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=/tmp/logrange-release-readiness/prefix
cmake --build /tmp/logrange-release-readiness/quickstart-build --config Release --parallel 2
ctest --test-dir /tmp/logrange-release-readiness/quickstart-build -C Release --output-on-failure
```

Separate minimal CMake consumers using `find_package(LogRange CONFIG REQUIRED)`
and `target_link_libraries(app PRIVATE LogRange::logrange)`, and using
`add_subdirectory`, were each configured, built, and run successfully against
that prefix/source tree.

## Release status

There are no code or packaging blockers from this audit. Before any release,
the maintainer must explicitly authorize it and reconcile the changelog's
`1.0.0` release heading with the absence of a corresponding tag or GitHub
Release. The research README's `SameMinorVersion` wording should also be made
consistent with this package's `SameMajorVersion` metadata.

### Non-blocking recommendations

1. Protect `main` with the GCC, Clang, and MSVC CI checks required before
   merge, and limit workflow permissions to read-only unless a release job
   needs more.
2. Add scheduled research-repository stress/adversarial validation; keep it
   out of normal runtime PR CI.
3. After explicit authorization, automate a release-package smoke check and
   distribution workflow with least-privilege publishing credentials.

No Git tag, GitHub Release, package publication, or release artifact was
created by this audit.
