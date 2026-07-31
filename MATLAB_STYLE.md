# MATLAB Style Guide

Conventions for writing MATLAB code across projects. This guide is project-neutral and intended as a public baseline. It is informed by practical MATLAB code review experience and the MathWorks MATLAB Coding Guidelines v1.0 (2025-04-30). Where guidance differs, the rules below define the preferred style for code that adopts this guide.

## 1. Project layout

```text
src/
  +pkg/                  <- top-level namespace
    +subpkg/             <- nest by domain, not by file type
      MyClass.m
      myFunction.m
  Main.m                 <- thin entry points at src root
test/
  +test/
    +unit/               <- mirrors src/ namespacing under here
      +pkg/+subpkg/
        tMyClass.m       <- one test file per unit, "t" prefix
    +system/             <- cross-module / integration tests, same mirror
      +pkg/+subpkg/
        tMyClass.m       <- only when system-level coverage is needed
doc/                     <- markdown only, no generated HTML
deploy/                  <- packaging scripts and .prj artifacts
MyProject.prj            <- canonical project file at repo root
```

Rules:
- Group by domain (`+io`, `+model`, `+analysis`), not by kind (`+utils` and `+helpers` are smells).
- No `private/` folders. Encapsulate with `SetAccess = protected` instead; it survives refactors better.
- Tests live in a sibling `test/` tree, never alongside source. Mirror the `src/` package path under `+test/+unit/` always and `+test/+system/` when warranted.
- For `src/+pkg/+subpkg/MyClass.m`, the unit test lives at `test/+test/+unit/+pkg/+subpkg/tMyClass.m`, and the system test, if any, lives at `test/+test/+system/+pkg/+subpkg/tMyClass.m`.
- `resources/` is reserved for MATLAB project files. Put shipped templates, fixtures, and icons elsewhere, such as `templates/` or `test/fixtures/`.

## 2. Naming

| Kind | Style | Example |
|---|---|---|
| Class | `PascalCase` | `TaskQueue`, `MessageLevel` |
| Function (.m) | `camelCase` | `readConfig`, `extractComponents` |
| Local variable | `camelCase` | `progressDialog`, `cleanupObj` |
| Property | `PascalCase` | `IsComplete`, `MessageText` |
| Constant | `UPPER_SNAKE` inside a constants struct/class | `MAX_COUNT`, `DEG_TO_RAD` |
| Package | `lowercase`, prefixed `+` | `+model`, `+signal` |
| Enum member | `PascalCase` | `MessageLevel.Info` |
| Test file | `t` + unit name | `tTaskQueue.m` |
| Test method | `t` + behaviour | `tDefaultValues`, `tDisplayInfo` |

Rules:
- Do not use Hungarian prefixes such as `i_`, `m_`, or capital-`I` for interfaces. Type information belongs in `arguments` blocks.
- `snake_case` is reserved for adapter shims that wrap legacy external APIs. Keep the original symbol when cross-referencing upstream source is important.
- Acronyms are treated as words. In `camelCase`, an acronym at the start is fully lowercase and the next word's first letter capitalises normally (`htmlWrite`, not `htmlwrite` or `HTMLWrite`). Mid-name, either `createURL` or `createUrl` is acceptable. Pick one per project and stay consistent.
- Identifier names for variables, functions, classes, methods, and properties should be no longer than 32 characters. If you need more, you are probably describing the implementation rather than the role.
- Avoid shadowing names already on the MATLAB path, such as `sin`, `rand`, or `sqrt`.
- `UPPER_SNAKE_CASE` for constants is a deliberate deviation from the MathWorks "no underscores" rule. It makes constants visibly distinct from variables at the call site (`Constants.MAX_COUNT` vs `maxCount`).

## 3. Function and method signatures

Every public function and method declares an `arguments` block. No exceptions for new code.

```matlab
function [item, cleanupObj] = addItem(queue, level, nvp)
    arguments
        queue (1,:) myapp.TaskQueue
        level (1,1) myapp.MessageLevel = myapp.MessageLevel.Info
        nvp.Identifier (1,1) string = ""
        nvp.Message (1,1) string = ""
    end
    ...
end
```

Rules:
- Positional arguments first, with no more than 3, then a single `nvp` struct for everything else.
- Always declare size and class: `(1,1) string`, `(1,:) double`, `(:,:) myPkg.Thing`.
- Defaults belong in the `arguments` block, never in the body.
- Use validation functions such as `{mustBePositive}`, `{mustBeMember(x, ["a", "b"])}`, and `{mustBeScalarOrEmpty}` over hand-rolled checks.
- Output names describe the value, not the type: `item`, not `outStruct`.
- Output count should be no more than 4. If you need more, return a struct or a value class.
- Put `end` on every function, including local functions and nested functions. Implicit-end form is forbidden for new code.
- Name-value names use `UpperCamelCase`, such as `LineWidth`, not `lineWidth`. Pass them with `Name=Value` syntax at call sites, not `"Name", Value`.

## 4. Strings, numbers, collections

- Use `string` by default. Char arrays (`'...'`) are only for interop with functions that require them, such as `sprintf` format strings or some toolbox APIs.
- Build string arrays with `[]`, not `{}`: `msgs = ["a"; "b"; "c"]`.
- Prefer structs and string-keyed structs to `containers.Map`. Reach for `dictionary` only when keys are dynamic and large.
- Numeric defaults are typed: `(1,1) double = 0`, not `= 0`.
- Never use `==` or `~=` on floating-point values. Use `abs(a - b) < tol` or `isapprox` in releases where it is available. Integer comparisons with `==` are fine.
- Write floating-point literals with a leading zero: `0.1`, not `.1`.

## 5. Object-oriented patterns

- Use value classes for plain data, parameter bundles, and DTOs.
- Use handle classes for anything with identity, observers, or lifecycle, such as queues, UI controllers, and service objects.
- Properties default to `SetAccess = protected`. Public set access is opt-in and rare.
- Use `SetObservable` deliberately. It enables `waitfor` and `listener`; document the reason at the property.
- Events are declared with `(NotifyAccess = protected)` and fired with `notify(obj, "EventName")`.
- Abstract base classes use the `Abstract` modifier and live next to their concrete subclasses. Do not use a capital-`I` interface convention.
- For owner-bound child objects, use weak owner references when callbacks, listeners, or children need to refer back to their owner. Make lifecycle explicit: construction may register the child, but setup that depends on sibling objects or a complete object graph belongs in a post-construction hook such as `initialize`.
- Keep coordinator objects narrow. They should sequence work, apply policy, or suppress/retry phases; they should not gather every collaborator and drive each domain object's internals. Prefer methods that let the owning object resolve its normal collaborators. Pass collaborators explicitly when that is the public contract or when it makes the unit easy to test with mocks or alternate implementations.
- Keep implementation helpers behind the object that owns them. Name private collaborators by role, such as `Engine`, `Strategy`, `Adapter`, and `Store`. Reserve `Controller` for an object that exposes a meaningful control surface or coordinates behavior across a boundary.
- Avoid `Sealed`. Default is unsealed. Only seal a class when there is a concrete reason, such as custom indexing semantics or a security boundary. Sealed-by-default is a refactor tax.
- Validate properties through the property block's size/class validators and `{mustBe...}` validators, not through a `set.Foo` method. Reserve `set.Foo` methods for transformation, not validation.
- Avoid `get.Foo` methods for non-dependent properties. Use dependent properties only when the value is computed from other properties, backward compatibility requires an alias, or the getter triggers a deliberate side effect.
- Close blocks with trailing comments only for long class definitions: `end % methods`, `end % classdef`.

## 6. Loops and control flow

- Do not grow arrays in a loop. Preallocate with `zeros`, `strings`, `repmat(seed, 1, n)`, or `createArray`. For complex classes where the seed instance is awkward to construct, collect into a cell array and concatenate at the end:
  ```matlab
  tmp = cell(1, n);
  for k = 1:n
      tmp{k} = MyClass(...);
  end
  result = [tmp{:}];
  ```
  This is often faster than `repmat`-then-overwrite for non-trivial constructors.
- Do not modify the loop iterator inside a `for` loop body. If you need to skip or reorder, use a `while` loop or filter the data first.
- `switch` always has an `otherwise` branch. If `otherwise` is unreachable by construction, leave a one-line comment explaining why.
- Prefer early `return` and `continue` to deeper nesting. Guard clauses at the top of a function are clearer than wrapping the body in nested conditionals. This diverges from the MathWorks preference to minimise `break`, `continue`, and `return`.
- Limit nesting of loops and conditionals to 5 levels. If you hit that, extract a helper.

## 7. Resource lifecycle

- Reach for `onCleanup` whenever a function acquires a resource, such as a file handle, dialog, or transient state. It survives `error` and early `return`.
- Use `matlab.lang.WeakReference` when a closure or child object needs to refer back to its owner. This avoids garbage-collection cycles on handle classes.
- When iterating to release handles, guard each element in `try/catch`; handle-array teardown is order-sensitive.

## 8. Errors

- Always throw errors with a colon-delimited identifier matching the package path: `error("pkg:unit:reason", "Human message: %s", detail)`.
- `try/catch` is for converting errors, such as rethrowing as a different identifier or wrapping in a status object, not for swallowing them. An empty catch block needs a comment explaining why.
- Wrap caught exceptions in `MException` directly. Do not maintain a custom exception hierarchy by default.
- `assert` is reserved for invariants inside a function. Boundary validation belongs in `arguments`.
- `throwAsCaller` is allowed and preferred in validation wrappers where the public method should appear as the error origin. This diverges from MathWorks guidance.
- Error messages should state the problem, then the fix where one exists: `"RelTol must be a nonnegative scalar"`, not `"bad arg"`.

## 9. Comments

- Public-facing functions, such as `run`, `launch`, or anything users call directly, get a help block: one-line H1 line, then inputs and outputs. Internal functions get a single one-line header only when useful.
- Inline comments explain why, not what. If you find yourself describing what a line does, rename the variable instead.
- Leave a short comment where ownership or lifecycle is split across objects. State the boundary: who owns lifetime, who owns placement/composition, and who adapts events or behavior.
- `%% Section` markers are for tests and scripts only. Production `.m` files do not need them; that is what folders and classes are for.
- No tag comments referencing PRs, tickets, or "added for X". Those rot.

## 10. Tests

- Framework: `matlab.unittest`, with `matlab.uitest.TestCase` for app/UI tests. Never use script-style tests.
- File `tFoo.m` tests unit `Foo`. Method `tBehaviour` describes one behaviour.
- Unit tests live under `test/+test/+unit/` mirroring the source package path. System tests live under `test/+test/+system/` likewise.
- Use built-in fixtures such as `WorkingFolderFixture` and `PathFixture` rather than `addpath` or `cd` in setup.
- Parameterise with `properties (TestParameter)`; one struct field per case.
- For function shadowing and mocking, prefer a fixture-based mocking pattern. Do not monkey-patch by hand.
- Use `TestClassSetup` for expensive one-off work and `TestMethodSetup` for per-test isolation.

## 11. Config and entry points

- App config lives in `config.json` at the repo root, with `config.template.json` checked in. Load via a single `getConfig` helper that fails loudly when the file is missing.
- Use one `.prj` file per repo, at the root, named after the project.
- Keep `src/Main.m` or an equivalent entry point under about 30 lines: parse args, build the top-level object, and hand off.

## 12. Formatting and whitespace

- Indentation: 4 spaces, never tabs.
- Line length should be no more than 120 characters. Break long lines after a comma, after a space, or at a binary operator.
- One statement per line.
- No trailing whitespace.
- One space either side of binary operators: `=`, `<`, `>=`, `&&`, `||`, `+`, `-`, and so on. This diverges from the MathWorks "spaces around `+`/`-` only on main terms" rule because that rule is too subjective to enforce.
- No spaces around `=` in `Name=Value` syntax: `plot(x, y, LineWidth=2)`.
- No spaces around `*`, `/`, `\`, `^`, and their element-wise variants.
- No spaces around the colon operator: `2:2:10`, not `2 : 2 : 10`.
- No spaces immediately inside `(...)`, `[...]`, or `{...}`.
- Space after `,` and `;` except at end of line.
- One blank line between local functions, between method declarations, and between separate method/property blocks in a `classdef`. No blank lines at the start or end of a file.
- No alignment padding. One space between identifier, size, type, and default. Aligning columns with extra spaces creates noisy diffs every time a longer name is added.
  ```matlab
  % Yes
  Signals (1,:) myapp.model.SignalSpec = myapp.model.SignalSpec.empty(1,0)
  Results (1,:) myapp.model.MappingResult = myapp.model.MappingResult.empty(1,0)
  RulesPath (1,1) string = ""

  % No - padded for column alignment
  Signals   (1,:) myapp.model.SignalSpec = ...
  Results   (1,:) myapp.model.MappingResult       = ...
  RulesPath (1,1) string = ""
  ```
- Size tuples are tight: `(1,1)`, `(1,:)`, `(:,:)`, `empty(1,0)`. Do not put spaces after the comma inside size specifiers. The general "space after comma" rule applies to function argument lists and array literals, not to size tuples.

## 13. Things we do not do

- `global` variables.
- `eval`, `evalin`, or `assignin` outside of test fixtures.
- `clear all`, `close all`, or `clc` in committed code.
- Suppressing warnings without a comment explaining which warning and why.
- Re-exporting symbols just to shorten an import path; use the full package name.
- `Sealed` classes by default. Only use them when justified.
- `varargin` or `varargout` to fake name-value arguments. Use a proper `arguments` block.
- Command-form syntax, such as `load foo.mat` or `clear x`, inside functions or methods. Use functional form only.
- `addpath` or `rmpath` inside functions without resetting on exit. Use `onCleanup`.
