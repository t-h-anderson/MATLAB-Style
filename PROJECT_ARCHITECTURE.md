# Project Architecture Guide

Repository and project-structure conventions for MATLAB projects that adopt this
style guide.

## Repository layout

```text
src/
  +pkg/                  <- top-level namespace
    +subpkg/             <- nest by domain, not by file type
      MyClass.m
      myFunction.m
  +util/                 <- true reusable utilities, used sparingly
  Main.m                 <- thin entry points at src root
test/                    <- separate test tree; see TESTING_AND_VERIFICATION.md
models/                  <- Simulink models and supporting model artifacts
extern/                  <- third-party source, vendored tools, and dependencies
artefacts/               <- generated local outputs; omitted from git
templates/               <- starter files copied into adopting projects
doc/                     <- markdown only, no generated HTML
deploy/                  <- packaging scripts and deploy-only artifacts
resources/               <- MATLAB project infrastructure files
dependencies.txt         <- dependency inventory and source locations
MyProject.prj            <- canonical project file at repo root
```

Rules:
- Group by domain (`+io`, `+model`, `+analysis`), not by kind (`+utils` and `+helpers` are smells).
- Avoid `private/` folders for new object-oriented code. Encapsulate class internals with access attributes such as
  `SetAccess = protected`; use package-scoped functions when helper functions need to be shared inside a package.
- Tests live in a sibling `test/` tree, never alongside source. Test layout and naming are defined in
  [Testing And Verification](TESTING_AND_VERIFICATION.md).
- `resources/` is reserved for MATLAB project infrastructure files, especially pre-R2027a MATLAB project metadata. Treat
  it as maintained by MATLAB project infrastructure, not by hand.
- Put template artifacts under `templates/`. Put fixtures, icons, and vendored assets under `test/`, `extern/`, or
  another domain-specific folder.

## Source organization

- Put production MATLAB code under `src/`.
- Use `+pkg` package folders for namespaces and nest packages by domain.
- Use `src/+util/...` only for true utilities that could reasonably be reused between projects. Prefer domain packages
  for project-specific helpers.
- Utility names are more likely to clash with MATLAB, toolbox, or project symbols. Keep them specific and check for
  name collisions before adding a new utility.
- Keep class folders and package folders valid MATLAB layouts. Do not mix package paths, class folders, and scripts in
  ways that require ad hoc path manipulation.
- Keep `src/Main.m` or an equivalent entry point under about 30 lines: parse args, build the top-level object, and hand
  off.
- Put Simulink models under `models/`; see [Simulink Style](SIMULINK_STYLE.md).
- Put third-party source, vendored tools, and external dependencies under `extern/`.
- Keep generated outputs under `artefacts/` and omit them from git unless a concrete publishing requirement exists.
- Track dependencies in `dependencies.txt`. Enumerate each dependency and where it comes from.
- Seed standard project files from `templates/` when the guide provides a template, such as
  `templates/LOCAL_STYLE_OVERRIDES.md` or `templates/dependencies.txt`.

## Config and project files

- App config lives in `config.json` at the repo root, with `config.template.json` checked in.
- Load config via a single `getConfig` helper that fails loudly when the file is missing.
- Use one `.prj` file per repo, at the root, named after the project.
- Deploy-only project files or package metadata may live under `deploy/` when they are not the canonical development
  project.
- Use `.mlapp` and App Designer for prototypes only. Production apps should move behaviour into testable MATLAB classes
  and functions.

## Test artifacts

- Put shared test artifacts under `test/artefacts/`.
- Put artifacts that are specific to one test next to that test.
- Keep source fixtures and expected data out of `resources/`; that folder is for MATLAB project infrastructure.

## Documentation and deploy artifacts

- Keep human-authored documentation under `doc/`.
- Do not commit generated HTML documentation unless the project has a concrete publishing requirement.
- Keep packaging scripts and `.prj` artifacts under `deploy/`.
- Keep build and external-tool workflow guidance in [Tooling](TOOLING.md), not in source packages.
