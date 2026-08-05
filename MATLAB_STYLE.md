# MATLAB Style Guide

Conventions for writing MATLAB code across projects. This guide is project-neutral
and intended as a public baseline. It is informed by practical MATLAB code review
experience and the MathWorks MATLAB Coding Guidelines v1.0 (2025-04-30). Where
guidance differs, the rules here define the preferred style for code that adopts
this guide.

The guide is split by concern:

- [Project Architecture](PROJECT_ARCHITECTURE.md): repository layout, packages,
  project files, config, docs, deploy artifacts, and entry points.
- [Code Style And Architecture](CODE_STYLE_ARCHITECTURE.md): MATLAB naming,
  signatures, data types, object-oriented patterns, control flow, lifecycle,
  errors, comments, formatting, and banned patterns.
- [Testing And Verification](TESTING_AND_VERIFICATION.md): test layout,
  `matlab.unittest` conventions, source-test pairing, Code Analyzer, and
  structural checks.
- [Simulink Style](SIMULINK_STYLE.md): model layout, naming, and verification
  defaults for Simulink projects.
- [Tooling](TOOLING.md): optional build automation, GitHub/CI conventions,
  AI-assisted project customization, and other external workflow tools.
- [Local Style Overrides Template](templates/LOCAL_STYLE_OVERRIDES.md):
  project-local deviations from this guide.

Use the focused guide for the decision you are making. When rules overlap, the
more specific guide wins. Local overrides must be explicit and should be kept
small.
