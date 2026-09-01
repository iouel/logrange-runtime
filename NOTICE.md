# Provenance

This repository is the supported, user-facing runtime for LogRange: a
header-only C++17 library for signed log-domain accumulation.

## Source of extraction

The code here — `include/logrange/log_math.h`, the CMake install/package
support, the essential runtime tests, and the quickstart example — was
extracted from the research repository:

- **Repository:** [`iouel/logrange`](https://github.com/iouel/logrange)
- **Branch:** `projectsummary`
- **Commit:** `b288b150abbba4a2cc8cbfc63ef102724e7803f6`

That repository is the research, validation, and audit-trail source for this
library: the derivation of the error contract, the adversarial bound search
that tried (twice) to refute it, benchmark methodology and raw numbers, and
the diagnostic/rewrite tooling that is explicitly **not** part of this
product. Consult it for provenance, not for installation — this repository
stands alone for that.

## What was intentionally left behind

This repository does not include:

- `matcher/` — the LLVM-based diagnostic/scanner tooling
- `pass/` — the experimental LLVM rewrite pass prototype
- LLVM/bitcode/WSL-specific setup and CI
- raw matcher/study data and result corpora
- lab-notebook and audit documents (`TODO.md`, `logrange_intent.md`,
  `BASELINE.md`, and similar experimental narratives)
- research-only exploratory tests not required to protect the runtime's
  supported contract (`bound_search.cpp`, `chain_search.cpp`,
  `test_rescue_shim.cpp`, and the benchmark harness)

None of the above are claimed to be part of this product. If you need the
diagnostic scanner, the rewrite pass, or the full research record, go to
`iouel/logrange` (branch `projectsummary`).

## License

The extracted code carries its original Apache License, Version 2.0. See
[LICENSE](LICENSE).
