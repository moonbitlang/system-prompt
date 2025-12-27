---
name: moonbit-agent-guide
description: Guide for writing, refactoring, and testing MoonBit projects. Use when working in MoonBit modules or packages, organizing MoonBit files, using moon tooling (build/check/test/doc/ide), or following MoonBit-specific layout, documentation, and testing conventions.
---

# MoonBit Agent Guide

## Scope and References

- Use this skill for project layout, file organization, tooling, and workflow guidance for MoonBit codebases.
- Read `references/language-guide.md` for detailed language syntax, types, patterns, and examples.

## Project Layout and File Organization

- Detect a module by `moon.mod.json`; run all `moon` subcommands from the module root.
- Detect a package by `moon.pkg.json`; treat each directory as a package and keep files cohesive.
- Packages without `moon.pkg.json` are not recognized.
- Module boundaries matter for dependency management and import paths.
- A package is the compilation unit; package name is defined in source, not file name.
- Imports always use module plus package paths; never refer to file names.
- Typical layout includes `moon.mod.json` at the module root, packages in subdirectories, and optional `cmd/main` executables with `is_main: true`.
- Package directories list dependencies in `moon.pkg.json` and may include `_test.mbt` and `_wbtest.mbt` files.
- Root packages often include `README.mbt.md` with an optional `README.md` symlink.
- Prefer many small, focused files; split large or unfocused files and group by feature.
- Add new code to an existing feature-matching file when possible; otherwise create a new focused file.
- Move declarations freely within a package; file names are organizational only.
- Any declaration in a package can reference any other declaration regardless of file.
- Avoid misc or util grab bags; choose descriptive file names per feature.
- Place tests in `*_test.mbt` files; use `*_wbtest.mbt` only for white-box needs.
- Multiple small test files are fine and encouraged.
- Separate top-level items with `///|` so tools can split files reliably.
- Keep deprecated blocks in `deprecated.mbt` per directory when applicable.

## API Discovery and Navigation

- Use `moon doc` first for API discovery and signatures; treat it as the primary lookup tool.
- Query `moon doc` with `@pkg`, `Type`, or `Type::method`; use `*` wildcards and empty query to list available packages/symbols.
- Use wildcards like `String::*rev*` to scan for partial matches.
- Use `moon doc "@pkg"` to list exported symbols and `moon doc ""` at module root to list packages.
- Empty `moon doc` shows packages at module root, symbols inside a package, and packages outside a module.
- Use `moon doc "[@pkg.]sym"` for values/functions, `moon doc "[@pkg.]Sym"` for types, and `moon doc "[@pkg.]T::sym"` for methods.
- Use `moon doc "TypeName"` to inspect type definitions and methods; use `moon doc "Type::method"` for method discovery.
- Use `moon doc "@pkg.fn_name"` before adding or changing call sites, then `moon ide find-references fn_name` to mirror existing usage.
- Use `moon ide outline`, `moon ide find-references`, and `moon ide peek-def` for symbol scans and cross-refs; prefer these over grep for outlines.
- Use `moon ide find-references Symbol` to locate usages when fixing tests or migrating APIs.
- `moon ide find-references` searches the current module.
- Use `rg` or `grep` only when outline output is insufficient or unavailable.
- Use `moon ide outline .` for a package outline and `moon ide outline file.mbt` for a file outline.
- Use `moon ide peek-def -symbol Name -loc path:line:col` for inline definition context; the line must be accurate.
- Read `.mbti` interfaces when you need a full public API snapshot or when offline.
- Start with `~/.moon/lib/core/builtin/pkg.generated.mbti` for core types; use `.mooncakes` for dependencies.
- Some builtin types (for example, `String`) expose APIs in both `builtin` and their dedicated packages.
- Run `moon info` in the project to regenerate package `.mbti` files and confirm public API changes; it only works for packages in the current project.

## Build and Configuration Essentials

- Configure dependencies and imports in `moon.pkg.json`; do not add imports in source files.
- Import format is `module_name/package_path`; call via `@alias.function()`.
- Call imported packages via `@alias.fn`; use aliases from `moon.pkg.json`.
- Use the last path segment as the default alias when one is not specified.
- Use an import object with `path` and `alias` to set custom aliases.
- Do not add standard library packages to dependencies or imports; use them directly (for example, `@strconv`).
- Do not use `moon add` for standard library packages; they are already available.
- If you see errors about `moonbitlang/core/...` imports, remove them from `moon.pkg.json`.
- Create a new package by adding a directory with `moon.pkg.json` and importing it where needed.
- Use `is_main` for executables; use `import`, `test-import`, and `wbtest-import` for dependency lists.
- Use `targets`, `link`, and `warn-list` in `moon.pkg.json` for conditional compilation and build config.
- Use `targets` conditions such as `wasm`, `wasm-gc`, `js`, `native`, `debug`, and `release` with `and`/`or`/`not`.
- Use `targets` as a map from file name to conditions (for example, `wasm_only.mbt: ["wasm"]`).
- Use nested arrays with `and`/`or`/`not` for complex target expressions.
- Use `link` as `true` or as a per-backend object; set exports and other backend settings there.
- Per-backend link settings include `wasm` exports/import memory, `wasm-gc` string settings, `js` format, and `native` compiler flags.
- Use `warn-list` in `moon.mod.json` or `moon.pkg.json`; use `moonc build-package -warn-help` to list warnings.
- `warn-list` values use a string like `-2-29` to disable specific warning numbers.
- Common warnings: 1 unused function, 2 unused variable, 11 partial match, 12 unreachable, 29 unused package.
- Use `pre-build` for code generation and embedding external assets; use `$input` and `$output` placeholders in commands.
- Use `:embed -i $input -o $output --name name --text` for simple text embedding.
- Use `moon.mod.json` for module metadata; `name` is `username/project`, `source` is optional, `version`/`repository`/`keywords`/`description` describe the module, and `deps` is for mooncakes packages.

## Common moon commands

- Use `moon new` to create projects, `moon build` to build, and `moon run cmd/main` to run executables.
- Use `moon check` for type checks, `moon fmt` for formatting, and `moon test` for tests.
- Use `moon check --target all` to type check all backends.
- Use `moon test --update` to refresh snapshots, `moon test -v` for verbose output, and `moon coverage analyze` for coverage.
- Use `moon add` (optionally with `@version`), `moon remove`, and `moon update` for dependency management.

## Documentation and README

- Write docstrings with `///` and delimit blocks with `///|`.
- Use `mbt check` blocks in docstrings only for tests; use `mbt` for non-test examples.
- Docstring `mbt check` blocks are type checked and run by `moon test --update`.
- `README.mbt.md` `mbt check` blocks are included as code and run by `moon check` and `moon test`.
- Generate `README.mbt.md` in the package directory; symlink to `README.md` if needed.
- Aim to cover at least 90% of public API surfaces with concise examples.
- Organize README by feature (construction, consumption, transformation, usage tips).

## Testing Workflow

- Prefer black-box tests using public APIs; use white-box tests only when necessary.
- Use the auto-imported package in `*_test.mbt` or `*_wbtest.mbt` via `@packagename`.
- Call public APIs via `@packagename.fn` in black-box tests.
- Use `inspect(...)` for snapshots; use `@json.inspect(...)` for complex values and derive `ToJson` where needed.
- Ensure types implement `Show` or `ToJson` so `inspect` or `@json.inspect` can render them.
- Prefer inspecting full return values when reasonable; update snapshots with `moon test --update`.
- If snapshot content is unknown, start with `inspect(value)` and run `moon test --update` or `moon test -u`.
- After updating snapshots, review test diffs; `content=` fields are updated automatically.
- Use `try?` to capture `raise` results in tests.
- Group related checks in one `test` block; prefix panic tests with `test "panic ..."`.
- If a panic test returns a value, wrap it with `ignore(...)` to avoid unused warnings.
- Run `moon test` and use `moon test --update` when snapshots change; use `moon test <dir>` or `moon test <file>` to scope runs.
- Run `moon fmt` after updating tests.

## Development Workflow

- Run `moon check` regularly; run `moon fmt` for formatting.
- Run `moon info` to verify public API diffs before finalizing changes.
- If `.mbti` outputs do not change, the refactor is typically safe for external users.
- Review `.mbti` diffs to confirm public API changes are expected.
- Prefer block-by-block edits over whole-file replacements.

## Spec-Driven Development

- Write a `spec.mbt` with `#declaration_only` for contracts and public signatures.
- The `spec.mbt` name is conventional, not mandatory; it can be read-only.
- Add `spec_*_test.mbt` files to validate the spec surface and run `moon check` to type check them.
- Use `#declaration_only` for types, methods, and functions; `pub type` is intentionally opaque.
- The spec file can also contain normal code; implement declarations in separate files within the same package.
- Run `moon test` to validate spec behavior.
