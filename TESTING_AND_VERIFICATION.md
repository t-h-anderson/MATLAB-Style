# Testing And Verification Guide

Testing and verification conventions for MATLAB projects that adopt this style
guide.

## Test layout

Tests live in a sibling `test/` tree, never alongside source. Mirror the `src/`
package path under `+test/+unit/` always and `+test/+system/` when warranted.

```text
test/
  +test/
    +unit/
      +pkg/+subpkg/
        tMyClass.m       <- one unit test file per source unit
    +system/
      +pkg/+subpkg/
        tMyClass.m       <- only when system-level coverage is needed
```

For `src/+pkg/+subpkg/MyClass.m`, the unit test lives at
`test/+test/+unit/+pkg/+subpkg/tMyClass.m`, and the system test, if any, lives at
`test/+test/+system/+pkg/+subpkg/tMyClass.m`.

## Source-test pairing

- Each production source file has a corresponding unit test file unless the file is a thin entry point, generated
  artifact, or explicitly documented exception.
- File `tFoo.m` tests unit `Foo`.
- Method `tBehaviour` describes one behaviour.
- Unit tests cover one source unit. System tests cover cross-module or integration behaviour.
- Shared test artifacts live under `test/artefacts/`. Test-specific artifacts live next to the test that owns them.

## MATLAB test style

- Framework: `matlab.unittest`, with `matlab.uitest.TestCase` for app/UI tests. Never use script-style tests.
- Use built-in fixtures such as `WorkingFolderFixture` and `PathFixture` rather than `addpath` or `cd` in setup.
- Parameterise with `properties (TestParameter)`; one struct field per case.
- For function shadowing and mocking, prefer a fixture-based mocking pattern. Do not monkey-patch by hand.
- Use `TestClassSetup` for expensive one-off work and `TestMethodSetup` for per-test isolation.
- `%% Section` markers are allowed in tests when they improve scanability.

## Verification checks

`runtests` alone is never the whole verification story when a project also
depends on structure, path setup, Code Analyzer, generated artifacts, or CI.

Recommended verification includes:
- MATLAB test execution for unit and relevant system tests.
- MATLAB Code Analyzer on production and test `.m` files.
- Structural checks for valid package layout, class-folder layout, project files, and root files required by the
  project.
- Checks for broad `try/catch` patterns, such as `catch` blocks that swallow `MException` without rethrowing,
  converting, or asserting a documented reason.
- Checks that MATLAB-only code remains in MATLAB-owned folders such as `src/`, `models/`, and `tools/`, rather than
  being duplicated in CI shell glue or external wrappers.
- Coverage checks for projects that collect coverage. The default target is at least 80% statement coverage unless the
  project documents a different target.

For projects with build automation, expose these checks through `buildtool`; see
[Tooling](TOOLING.md).
