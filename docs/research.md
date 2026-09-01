# Research and evidence

LogRange's supported runtime API, CMake package, examples, and focused
regression tests live in this repository. The separate
[`iouel/logrange`](https://github.com/iouel/logrange/tree/projectsummary)
repository, on its `projectsummary` branch, keeps the material that is not
needed to consume the library:

- numerical derivations and the detailed error-contract rationale;
- adversarial validation and experimental history;
- benchmark methodology and raw measurements;
- diagnostic matcher research and LLVM rewrite-pass prototypes.

For the exact source revision from which this runtime was extracted, see
[NOTICE.md](../NOTICE.md). The research repository is evidence and provenance,
not an additional installation requirement: use the CMake instructions in the
[README](../README.md) to consume LogRange.
