---
name: moonbit-agent-guide
description: Guide for writing, refactoring, and testing MoonBit projects. Use when working in MoonBit modules or packages, organizing MoonBit files, using moon tooling (build/check/test/doc/ide), or following MoonBit-specific layout, documentation, and testing conventions.
---

# MoonBit Project Layouts

MoonBit source files use the `.mbt` extension and interface files `.mbti`. At
the top-level of a MoonBit project there is a `moon.mod.json` file specifying
the metadata of the project. The project may contain multiple packages, each
with its own `moon.pkg.json` file.

Here are some typical project layouts you may encounter:

- **Module**: When you see a `moon.mod.json` file in the project directory, you
  are already in a MoonBit project.
  A MoonBit *module* is like a Go module.
  It is a collection of packages, usually corresponding to a repository or project.
  Module boundaries matter for dependency management and import paths.
  A module contains many packages in subdirectories.

- **Package**: When you see a `moon.pkg.json` file, but not a `moon.mod.json`
  file, it means you are in a MoonBit package. All subcommands of `moon` will
  still be executed in the directory of the module (where `moon.mod.json` is
  located), not the current package.
  A MoonBit *package* is the actual compilation unit (like a Go package).
  All source files in the same package are concatenated into one unit.
  The `package` name in the source defines the package, not the file name.
  Imports refer to module + package paths, NEVER to file names.

- **Files**:
  A `.mbt` file is just a chunk of source inside a package.
  File names do NOT create modules or namespaces.
  You may freely split/merge/move declarations between files in the same package.
  Any declaration in a package can reference any other declaration in that package, regardless of file.

## Coding/layout rules you MUST follow:

1. Prefer many small, cohesive files over one large file.
   - Group related types and functions into focused files (e.g. http_client.mbt, router.mbt).
   - If a file is getting large or unfocused, create a new file and move related declarations into it.

2. You MAY freely move declarations between files inside the same package.
   - Moving a function/struct/trait between files does not change semantics, as long as its name and pub-ness stay the same.
   - It is safe to refactor by splitting or merging files inside a package.

3. File names are purely organizational.
   - Do NOT assume file names define modules, and do NOT use file names in type paths.
   - Choose file names to describe a feature or responsibility, not to mirror type names rigidly.

4. When adding new code:
   - Prefer adding it to an existing file that matches the feature.
   - If no good file exists, create a new file under the same package with a descriptive name.
   - Avoid creating giant “misc” or “util” files.

5. Tests:
   - Place tests in dedicated test files (e.g. *_test.mbt) within the appropriate package.
     For a package, besides `*_test.mbt`files,`*.mbt.md`are also blackbox test files, the code block `mbt check` are treated as test cases, they serve both purposes: documentation and tests.      
     You may have `README.mbt.md` files with `mbt check` code examples, you can also symlink `README.mbt.md` to `README.md`
     to make it integrate better with GitHub.
   - It is fine—and encouraged—to have multiple small test files.

## `.mbti` Files - Package Interface Documentation

MoonBit interface files (`pkg.generated.mbti`) are compiler-generated summaries of each package's public API surface. They provide a formal, concise overview of all exported types, functions, and traits without implementation details.
They are generated using `moon info`, they are useful for code review, when you have a commit that does not change
public APIs, `pkg.generated.mbti` files will remain unchanged, so it is recommended to put `pk.generated.mbti` in VCS.
You can also use `moon doc @moonbitlang/core/strconv` to explore the public API of a package interactively.


# Best Practices and Reference

## Common Pitfalls to Avoid

1. **Don't use uppercase for variables/functions** - compilation error
2. **Don't forget `mut` for mutable fields** - immutable by default
3. **Don't assume value semantics** - most types pass by reference
4. **Don't ignore error handling** - errors must be explicitly handled
5. **Don't use `return` unnecessarily** - last expression is the return value
6. **Don't create methods without Type:: prefix** - methods need explicit type prefix
7. Don't forget to handle array bounds - use get() for safe access
8. Don't mix up String indexing (returns Int). Use `for char in s {...}` for char iteration
9. Don't forget @package prefix when calling functions from other packages
10. Don't use ++ or -- (not supported), use `i = i + 1` or `i += 1`
11. **Don't add explicit `try` for error-raising functions** - errors propagate automatically (unlike Swift)
12. **Legacy syntax**: Older code may use `function_name!(...)` or `function_name(...)?` - these are deprecated; use normal calls and `try?` for Result conversion

# MoonBit ToolChain Essentials

## Idiomatic Project Structure

MoonBit projects use `moon.mod.json` (module descriptor) and `moon.pkg.json`
(package descriptor):

```
my_module
├── Agents.md                 # Guide to Agents
├── README.mbt.md             # Markdown with tested code blocks (`test "..." { ... }`)
├── README.md -> README.mbt.md
├── cmd                       # Command line directory
│   └── main
│       ├── main.mbt
│       └── moon.pkg.json     # executable package with {"is_main": true}
├── liba/                     # Library packages
│   └── moon.pkg.json         # Referenced by other packages as `@username/my_module/liba`
│   └── libb/                 # Library packages
│       └── moon.pkg.json     # Referenced by other packages as `@username/my_module/liba/libb`
├── moon.mod.json             # Module metadata, source field(optional) specifies the source directory of the module
├── moon.pkg.json             # Package metadata (each directory is a package like Golang)
├── user_pkg.mbt              # Root packages, referenced by other packages as `@username/my_module`
├── user_pkg_wbtest.mbt       # White-box tests (only needed for testing internal private members, similar to Golang's package mypackage)
└── user_pkg_test.mbt         # Black-box tests
└── ...                       # More package files, symbols visible to current package (like Golang)
```

## Essential Commands

- `moon new my_project` - Create new project
- `moon run cmd/main` - Run main package
- `moon build` - Build project
- `moon check` - Type check without building, use it regularly
- `moon info` - Type check and generate `mbti` files
- `moon check --target all` - Type check for all backends
- `moon add package` - Add dependency
- `moon remove package` - Remove dependency
- `moon fmt` - Format code

### Test Commands

- `moon test` - Run all tests
- `moon test --update`
- `moon test -v` - Verbose output with test names
- `moon test [dirname|filename]` - Test specific directory or file
- `moon coverage analyze` - Analyze coverage

## Package Management

### Adding Dependencies

```bash
moon add moonbitlang/x        # Add latest version
moon add moonbitlang/x@0.4.6  # Add specific version
```

### Updating Dependencies

```bash
moon update                   # Update package index
```

## Key Configuration

### Module (`moon.mod.json`)

```json
{
  "name": "username/hello", // Required format for published modules
  "version": "0.1.0",
  "source": ".", // Source directory(optional, default: ".")
  "repository": "", // Git repository URL
  "keywords": [], // Search keywords
  "description": "...", // Module description
  "deps": {
    // Dependencies from mooncakes.io, using`moon add` to add dependencies
    "moonbitlang/x": "0.4.6"
  }
}
```

### Package (`moon.pkg.json`)

```json
{
  "is_main": true,                 // Creates executable when true
  "import": [                      // Package dependencies
    "username/hello/liba",         // Simple import, use @liba.foo() to call functions
    {
      "path": "moonbitlang/x/encoding",
      "alias": "libb"              // Custom alias, use @libb.encode() to call functions
    }
  ],
  "test-import": [...],            // Imports for black-box tests, similar to import
  "wbtest-import": [...]           // Imports for white-box tests, similar to import (rarely used)
}
```

Packages per directory, packages without `moon.pkg.json` are not recognized.

## Package Importing (used in moon.pkg.json)

- **Import format**: `"module_name/package_path"`
- **Usage**: `@alias.function()` to call imported functions
- **Default alias**: Last part of path (e.g., `liba` for `username/hello/liba`)
- **Package reference**: Use `@packagename` in test files to reference the
  tested package

**Package Alias Rules**:

- Import `"username/hello/liba"` → use `@liba.function()` (default alias is last path segment)
- Import with custom alias `{"path": "moonbitlang/x/encoding", "alias": "enc"}` → use `@enc.function()`
- In `_test.mbt` or `_wbtest.mbt` files, the package being tested is auto-imported

Example:

```mbt
///|
/// In main.mbt after importing "username/hello/liba" in `moon.pkg.json`
fn main {
  println(@liba.hello()) // Calls hello() from liba package
}
```

## Using Standard Library (moonbitlang/core)

**MoonBit standard library (moonbitlang/core) packages are automatically imported** - DO NOT add them to dependencies:

- ❌ **DO NOT** use `moon add` to add standard library packages like `moonbitlang/core/strconv`
- ❌ **DO NOT** add standard library packages to `"deps"` field of `moon.mod.json`
- ❌ **DO NOT** add standard library packages to `"import"` field of `moon.pkg.json`
- ✅ **DO** use them directly: `@strconv.parse_int()`, `@list.List`, `@array.fold()`, etc.

If you get an error like "cannot import `moonbitlang/core/strconv`", remove it from imports - it's automatically available.

## Creating Packages

To add a new package `fib` under `.`:

1. Create directory: `./fib/`
2. Add `./fib/moon.pkg.json`: `{}` -- Minimal valid moon.pkg.json
3. Add `.mbt` files with your code
4. Import in dependent packages:

   ```json
   {
     "import": [
        "username/hello/fib",
        ...
     ]
   }
   ```
For more advanced topics like `conditional compilation`, `link configuration`, `warning control`, and `pre-build commands`, see `references/advanced-moonbit-build.md`.



# Development Workflow

## MoonBit Tips

- MoonBit code is organized in files/block style.
  A package is composed of a list of files, their order does not matter,
  keep them separate so that it is easy to focus on critical parts.

  Each block is separated by `///|`, the order of each block is irrelevant too. You can process
  block by block independently.

  You are encouraged to generate code in a block-by-block manner.

  You are encouraged to search and replace block by block instead of
  replacing the whole file.

  You are encouraged to keep each file focused.

- SPLIT the large file into small files, the order does not matter.

- Try to keep deprecated blocks in file called `deprecated.mbt` in each
  directory.

- `moon fmt` is used to format your code properly.

- `moon info` is used to update the generated interface of the package
  **in current project**. Each package has a generated interface file `.mbti`,
  it is a brief formal description of the package. If nothing in `.mbti`
  changes, this means your change does not bring the visible changes to the
  external package users, it is typically a safe refactoring.
  **Note**: `moon info` will only work with packages in the current project, and
  therefore you cannot use `moon info` to generate interface for dependencies
  like standard library.

- So in the last step, you typically run `moon info && moon fmt` to update the
  interface and format the code. You also check the diffs of `.mbti` file to see
  if the changes are expected.

- You should run `moon test` to check the test is passed. MoonBit supports
  snapshot testing, so in some cases, your changes indeed change the behavior of
  the code, you should run `moon test --update` to update the snapshot.

- You can run `moon check` to check the code is linted correctly, run it
  regularly to ensure you are not in a messy state.

- MoonBit packages are organized per directory; each directory has a
  `moon.pkg.json` listing its dependencies. Each package has its files and
  blackbox test files (common, ending in `_test.mbt`) and whitebox test files
  (ending in `_wbtest.mbt`).

- In the toplevel directory, there is a `moon.mod.json` file describing the
  module and metadata.

## MoonBit Package `README` Generation Guide

- Output `README.mbt.md` in the package directory. 
  `*.mbt.md` file and docstring contents treats `mbt check` specially.
  `mbt check` block will be included directly as code and also run by `moon check` and `moon test`. 
  In docstrings, `mbt check` should only contain test blocks.
  If you are only referencing types from the package, you should use `mbt` which will only be syntax highlighted.
  Symlink `README.mbt.md` to `README.md` to adapt to systems that expect `README.md`. 
- Aim to cover ≥90% of the public API with concise sections and examples.
- Organize by feature: construction, consumption, transformation, and key usage tips.

## MoonBit Testing Guide

Practical testing guidance for MoonBit. Keep tests black-box by default and rely on snapshot `inspect(...)`.

- Black-box by default: Call only public APIs via `@package.fn`. Use white-box tests only when private members matter.
- **Snapshots**: Prefer `inspect(value, content="...")`. If unknown, write `inspect(value)` and run `moon test --update` (or `moon test -u`).
  - Use regular `inspect()` for simple values (uses `Show` trait)
  - Use `@json.inspect()` for complex nested structures (uses `ToJson` trait, produces more readable output)
  - It is encouraged to `inspect` or `@json.inspect` the whole return value of a function if
    the whole return value is not huge, this makes test simple. You need `impl (Show|ToJson) for YourType` or `derive (Show, ToJson)`.
- **Update workflow**: After changing code that affects output, run `moon test --update` to regenerate snapshots, then review the diffs in your test files (the `content=` parameter will be updated automatically).
- Grouping: Combine related checks in one `test "..." { ... }` block for speed and clarity.
- Panics: Name test with prefix `test "panic ..." {...}`; if the call returns a value, wrap it with `ignore(...)` to silence warnings.
- Errors: Use `try? f()` to get `Result[...]` and `inspect` it when a function may raise.
- Verify: Run `moon test` (or `-u` to update snapshots) and `moon fmt` afterwards.

### Docstring tests
````mbt check
///|
/// Get the largest element of a non-empty `Array`.
///
/// # Example
/// ```mbt check
/// test {
///   inspect(sum_array([1, 2, 3, 4, 5, 6]), content="21")
/// }
/// ```
///
/// # Panics
/// Panics if the `xs` is empty.
pub fn sum_array(xs : Array[Int]) -> Int {
  xs.fold(init=0, (a, b) => a + b)
}
````

The MoonBit code in docstring will be type checked and tested automatically.
(using `moon test --update`)
## Spec-driven Development

- The spec can be written in a readonly `spec.mbt` file (name is conventional, not mandatory) with stub code marked as declarations:

```mbt check
///|
#declaration_only
pub type Yaml

///|
#declaration_only
pub fn Yaml::to_string(y : Yaml) -> String raise {
  ...
}

///|
#declaration_only
pub fn parse_yaml(s : String) -> Yaml raise {
  ...
}
```

- Add `spec_easy_test.mbt`, `spec_difficult_test.mbt` etc to test the spec functions; everything will be type-checked(`moon check`).
- The AI or students can implement the `declaration_only` functions in different files thanks to our package organization.
- Run `moon test` to check everything is correct.

- `#declaration_only` is supported for functions, methods, and types.
- The `pub type Yaml` line is an intentionally opaque placeholder; the implementer chooses its representation.
- Note the spec file can also contain normal code, not just declarations.

# Semantics based CLI tools

## API Discovery and Code Navigation (`moon doc` + `moon ide`)

**CRITICAL**: `moon doc '<query>'` is your PRIMARY tool for discovering available APIs, functions, types, and methods in MoonBit. It is **more powerful and accurate** than `grep_search`, `semantic_search`, or any file-based searching tools. Always prefer `moon doc` over other approaches when exploring what APIs are available.

For project-local symbols and navigation, use `moon ide outline .` to scan a package, `moon ide find-references <symbol>` to locate usages, and `moon ide peek-def` for inline definition context. These tools avoid loading large files into context and save tokens.

### Example scenarios

- **Adding a new call site**: Use `moon doc "@pkg.fn_name"` to confirm the API signature, then `moon ide find-references fn_name` to mirror existing usage patterns in the codebase.
- **Tracing a type’s role**: Use `moon ide outline .` to find where a type is defined, then `moon ide find-references TypeName` to see how it flows through functions without opening large files.
- **Fixing a failing test**: Use `moon ide find-references` on the failing symbol to find the impacted functions, then `moon ide peek-def` to inspect the definition context quickly.
- **Migrating an API**: Use `moon doc "@pkg"` to list available alternatives, then `moon ide find-references OldType/OldFn` to update all usages consistently.
- **Exploring unfamiliar package**: Use `moon ide outline .` to map files in the package, then `moon doc "Type::method"` to discover methods before editing.

### Query Syntax

`moon doc` uses a specialized query syntax designed for symbol lookup:
- **Empty query**: `moon doc `

  - In a module: shows all available packages in current module
  - In a package: shows all symbols in current package
  - Outside package: shows all available packages

- **Function/value lookup**: `moon doc "[@pkg.]sym"`
  
- **Type lookup**: `moon doc "[@pkg.]Sym"`

- **Method/field lookup**: `moon doc "[@pkg.]T::sym"`

- **Package exploration**: `moon doc "@pkg"`
  - Show package `pkg` and list all its exported symbols
  - Example: `moon doc "@json"` - explore entire `@json` package
  - Example: `moon doc "@encoding/utf8"` - explore nested package

### Workflow for API Discovery

1. **Finding functions**: Use `moon doc "@pkg.function_name"` before grep searching
2. **Exploring packages**: Use `moon doc "@pkg"` to see what's available in a package
3. **Method discovery**: Use `moon doc "Type::method"` to find methods on types
4. **Type inspection**: Use `moon doc "TypeName"` to see type definition and methods
5. **Package exploration**: Use `moon doc ""` at module root to see all available packages, including dependencies and stdlib
6. **Globbing**: Use `*` wildcard for partial matches, e.g. `moon doc "String::*rev*"` to find all String methods with "rev" in their name
### Examples

````bash
# search for String methods in standard library:
$ moon doc "String"

type String

  pub fn String::add(String, String) -> String
  pub fn String::at(String, Int) -> Int
  # ... more methods omitted ...

# list all symbols in a standard library package:
$ moon doc "@buffer"
moonbitlang/core/buffer

fn from_array(ArrayView[Byte]) -> Buffer
fn from_bytes(Bytes) -> Buffer
# ... more functions omitted ...

# list the specific function in a package:
$ moon doc "@buffer.new"
package "moonbitlang/core/buffer"

pub fn new(size_hint? : Int) -> Buffer
  Creates a new extensible buffer with specified initial capacity. If the
   initial capacity is less than 1, the buffer will be initialized with capacity
   1.
# ... more details omitted ...

$ moon doc "String::*rev*"  
package "moonbitlang/core/string"

pub fn String::rev(String) -> String
  Returns a new string with the characters in reverse order. It respects
   Unicode characters and surrogate pairs but not grapheme clusters.

pub fn String::rev_find(String, StringView) -> Int?
  Returns the offset (charcode index) of the last occurrence of the given
   substring. If the substring is not found, it returns None.

# ... more details omitted ...

**Best practice**: When implementing a feature, start with `moon doc` queries to discover available APIs before writing code. This is faster and more accurate than searching through files.

````

### `moon ide peek-def` for Definition Lookup

Use this when you want inline context for a symbol without jumping files.

``` file src/parse.mbt
L45:|///|
L46:|fn Parser::read_u32_leb128(self : Parser) -> UInt raise ParseError {
L47:|  ...
...:| }
```

```bash
$ moon ide peek-def -symbol Parser -loc src/parse.mbt:46:4
Definition found at file src/parse.mbt
  | ///|
2 | priv struct Parser {
  |             ^^^^^^
  |   bytes : Bytes
  |   mut pos : Int
  | }
  | 
  | ///|
  | fn Parser::new(bytes : Bytes) -> Parser {
  |   { bytes, pos: 0 }
  | }
  | 
  | ///|
  | fn Parser::eof(self : Parser) -> Bool {
  |   self.pos >= self.bytes.length()
  | }
  | 
```
For the `-loc` argument, the line number must be precise; the column can be approximate since `-symbol` helps locate the position.

### `moon ide outline` and `moon ide find-references` for Package Symbols

Use this to scan a package or file for top-level symbols and locate usages without opening files.

- `moon ide outline dir` outlines the current package directory (per-file headers)
- `moon ide outline parser.mbt` outlines a single file
- Useful when you need a quick inventory of a package, or to find the right file before `goto-definition`
- `moon ide find-references TranslationUnit` finds all references to a symbol in the current module

```bash
$ moon ide outline .
spec.mbt:
 L003 | pub(all) enum CStandard {
        ...
 L013 | pub(all) struct Position {
        ...
```

```bash
$ moon ide find-references TranslationUnit
```

# MoonBit Language Tour

## MoonBit-unique rules to keep in mind

- Keep `///|` separators between top-level items; edit block-by-block.
- Do not add `import` in `.mbt` files; declare imports in `moon.pkg.json` and call via `@alias.fn`.
- Respect visibility nuance: `pub` exposes read/pattern-match only, `pub(all)` allows construction, `pub(open)` allows external impls.
- Treat errors explicitly: declare `raise`, let errors propagate by default, and use `try?` or `try { } catch { }` only when handling.
- Remember control flow is expression-based; `if`, `match`, `while` return values, and `loop`/`for` have functional forms.
- Use `...` as a valid placeholder for stubs or incomplete implementations.

## MoonBit Error Handling (Checked Errors)

MoonBit uses checked error-throwing functions, not unchecked exceptions. All errors are subtype of `Error`, we can declare our own error types by `suberror`.
Use `raise` in signatures to declare error types and let errors propagate by
default. Use `try?` to convert to `Result[...]` in tests, or `try { } catch { }`
to handle errors explicitly.

```mbt check
///|
/// Declare error types with 'suberror'
suberror ValueError String

///|
/// Tuple struct to hold position info
struct Position(Int, Int) derive(ToJson, Show, Eq) 

///|
/// ParseError is subtype of Error
pub(all) suberror ParseError {
  InvalidChar(pos~:Position, Char) // pos is labeled
  InvalidEof(pos~:Position)
  InvalidNumber(pos~:Position, String)
  InvalidIdentEscape(pos~:Position)
} derive(Eq, ToJson, Show)

///|
/// Functions declare what they can throw
fn parse_int(s : String, position~ : Position) -> Int raise ParseError {
  // 'raise' throws an error
  if s is "" {
    raise ParseError::InvalidEof(pos=position)
  }
  ... // parsing logic
}

///|
/// Just declare `raise` to not track specific error types
fn div(x : Int, y : Int) -> Int raise {
  if y is 0 {
    raise Failure("Division by zero")
  }
  x / y
}

///|
test "inspect raise function" {
  let result : Result[Int, Error] = try? div(1, 0) 
  guard result is Err(Failure(_)) else {
    fail("Expected error")
  }  
}

// Three ways to handle errors:

///|
/// Propagate automatically
fn use_parse(position~: Position) -> Int raise ParseError {
  let x = parse_int("123", position=position)
  // Error auto-propagates by default.
  // Unlike Swift, you do not need to mark `try` for functions that can raise
  // errors; the compiler infers it automatically. This keeps error handling
  // explicit but concise.
  x * 2
}

///|
/// Mark `raise` for all possible errors, do not care which error it is.
/// For quick prototypes, `raise` is acceptable.
fn use_parse2(position~: Position) -> Int raise {
  let x = parse_int("123", position=position)
  x * 2
}

///|
/// Convert to Result with try?
fn safe_parse(s : String, position~: Position) -> Result[Int, ParseError] {
  let val1 : Result[_] = try? parse_int(s, position=position) // Returns Result[Int, ParseError]
  // try! is rarely used - it panics on error, similar to unwrap() in Rust
  // let val2 : Int = try! parse_int(s) // Returns Int otherwise crash

  // Alternative explicit handling:
  let val3 = try parse_int(s, position=position) catch {
    err => Err(err)
  } noraise { // noraise block is optional - handles the success case
    v => Ok(v)
  }
  ...
}

///|
/// Handle with try-catch
fn handle_parse(s : String, position~: Position) -> Int {
  try parse_int(s, position=position) catch {
    ParseError::InvalidEof => {
      println("Parse failed: InvalidEof")
      -1 // Default value
    }
    _ => 2
  }
}
```

Important: When calling a function that can raise errors, if you only want to
propagate the error, you do not need any marker; the compiler infers it.

For deeper syntax, types, and examples, read `references/moonbit-language-fundamentals.mbt.md`.
