# Tooling Guide

Optional conventions for build automation, CI, GitHub workflows, AI-assisted
projects, and other external tools. These rules apply only when the project uses
the relevant tool.

## Build automation

Projects that adopt automation should use a root `buildfile.m` as the canonical
entry point. CI, local checks, and agent workflows call `buildtool`; they do not
call scattered scripts directly.

Recommended tasks:
- `checkMlProject`: validates project structure, path setup, required files, and class/package layout.
- `checkCodeAnalysis`: runs MATLAB Code Analyzer on production and test code.
- `test`: runs the MATLAB test suite and depends on the relevant checks.
- `coverage`: runs or reports coverage when the project collects coverage.

Build helper code should live outside production packages, for example:

```text
tools/
  +buildtool/
    checkProject.m
    checkCodeAnalysis.m
    collectSourceFiles.m
```

Helper functions under `tools/+buildtool/` should follow the normal MATLAB code
style rules when they are committed source.

## GitHub and CI

- CI should call the same `buildtool` tasks that developers run locally.
- Keep CI-specific shell or YAML glue thin. Put MATLAB verification logic in MATLAB functions that can be run locally.
- Do not let CI be the only place a required project check exists.
- Generated reports and coverage artifacts should be written under `artefacts/` and omitted from git unless the project
  has a concrete publishing requirement.
- The default merge gate is passing tests plus at least 80% statement coverage when coverage is collected. Projects may
  document a different threshold when there is a clear reason.

## AI-assisted projects

Agent-assist is optional. Do not require these files in projects that do not use
coding agents.

Projects that use coding agents may keep their agent instructions versioned with
the code they regulate:
- `AGENTS.md`: project-specific contract and local deviations only.
- `.agents/agents.toml`: roles, permissions, and workflow limits.
- `.codex/skills/<project-skill>/SKILL.md`: project-local reusable skill instructions.

Rules:
- Reference shared guide files from agent instructions. Never duplicate the rules.
- Put project-specific deviations in `AGENTS.md`, not in the shared style guide.
- Treat role configuration as permissions, not promises. For example, a maker role may edit but not commit, while a
  verifier role may be read-only and produce blocking findings.
- Keep agent checks wired through the same `buildtool` tasks used by humans and CI whenever practical.

## Other external tools

- Keep external-tool configuration at the repository root unless the tool has a standard subfolder convention.
- Prefer small wrappers that call MATLAB entry points over duplicating MATLAB logic in another language.
- Document required external tools in the README or project setup docs.
- Maintain `dependencies.txt` at the repository root. For each dependency, list the name, purpose, source, version or
  commit when known, and whether it is vendored under `extern/` or installed separately.
- Use files under `templates/` as starter artifacts only. Copy them into the adopting project root before editing them.
