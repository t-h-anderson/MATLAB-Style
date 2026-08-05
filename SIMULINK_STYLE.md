# Simulink Style Guide

Short conventions for Simulink content. Most model-design details are TBD until
the guide has real project examples.

## Layout

- Put Simulink models and supporting model artifacts under `models/`.
- Keep generated simulation, coverage, and report outputs under `artefacts/` and
  omit them from git.
- Keep MATLAB support code in `src/` or `tools/`, not inside model folders.

## Naming

- Use descriptive model and subsystem names.
- Avoid names that differ only by case, spacing, or punctuation.
- TBD: signal, bus, Stateflow, data dictionary, and library naming conventions.

## Verification

- Simulink projects should expose model checks through the same `buildtool`
  workflow as MATLAB code.
- TBD: default Model Advisor profile, harness layout, Test Manager layout, and
  coverage expectations.
