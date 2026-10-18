# AGENTS.md

Guidance for AI agents working in this repository.

## What this repo is

Source for the book "[Crafting Interpreters](http://craftinginterpreters.com)" by Bob Nystrom. It contains:

- The book text as Markdown (`book/`, one file per chapter).
- **jlox** — the tree-walk interpreter in Java (`java/com/craftinginterpreters/lox/`).
- **clox** — the bytecode VM in C (`c/`).
- A Dart-based build/test toolchain (`tool/`) that weaves book text + code snippets into the static site (`site/`, generated but committed).
- A Lox test suite (`test/`, ~600 `.lox` scripts).

## Layout

| Path | Contents |
|---|---|
| `book/` | Markdown chapters; title-case `as-kebab-case.md` |
| `java/com/craftinginterpreters/lox/` | jlox sources (`Lox.java` is the entry point) |
| `java/com/craftinginterpreters/tool/` | `GenerateAst.java` — generates `Expr.java`/`Stmt.java` |
| `c/` | clox sources (`main.c` entry; chunk/compiler/scanner/vm/object/table/value/memory/debug) |
| `test/` | `.lox` tests grouped by feature; expectations in comments |
| `tool/bin/` | Dart scripts: `build.dart`, `test.dart`, `split_chapters.dart`, `compile_snippets.dart` |
| `util/c.make`, `util/java.make` | Sub-makes invoked by the root `Makefile` |
| `asset/` | Site templates (mustache) and styles (sass) |
| `site/` | Generated HTML — committed to git, regenerate with `make book` |
| `note/` | Author's working notes (answers to exercises, style guide, etc.) |
| `build/`, `gen/` | Build output and per-chapter split sources — generated, not edited |

## Prerequisites & environment

- Dart SDK (`make get` runs `dart pub get` in `tool/`), `javac`/`java`, and a C compiler (`cc`).
- The `Makefile` is POSIX-oriented (uses `find`, `cp`, `@` silencing, `diff`). This checkout is on **Windows (D:\Github\...)** — run make from Git Bash/MSYS2/WSL, and expect `.exe` naming nuances.
- `./clox` and `./jlox` at the repo root are convenience wrapper scripts created by the build.

## Common commands

```sh
make get            # one-time: dart pub get for tool/
make                # build site + clox + jlox
make clox           # release build of C interpreter -> build/clox + ./clox
make jlox           # generates AST classes, then compiles Java -> build/java/
make debug          # debug build of clox -> build/cloxd
make test           # run test suite against final clox and jlox
make test_clox      # ...clox only (also: test_jlox, test_c, test_java, test_all)
make book           # regenerate site/ in one batch
make serve          # dev server rooted at site/, rebuilds changed sources per request
```

Run the test runner directly for finer control: `dart tool/bin/test.dart <interpreter> [filter]`.

## Critical conventions — read before editing

1. **Snippet-weaving comments are load-bearing.** Both `c/` and `java/` sources are full of markers like `//> Chapter Name snippet-id` / `//< Chapter Name snippet-id` (and `/*> ... */`). `tool/bin/build.dart` uses them to splice code into the book, and `split_chapters.dart` uses them to generate per-chapter versions under `gen/`. Do not delete, rename, or reorder them; keep them paired and nested correctly.

2. **`Expr.java` and `Stmt.java` are generated.** Edit the grammar in `java/com/craftinginterpreters/tool/GenerateAst.java`, then run `make generate_ast` (part of `make jlox`). Never hand-edit the generated files. (`AstPrinter.java`, `RpnPrinter.java` are hand-written.)

3. **Warnings are errors.** C builds with `-std=c99 -Wall -Wextra -Werror` (release adds `-O3 -flto`; also buildable as C++11 via `make cpplox`). Java compiles with `-Werror`. Fix warnings, don't suppress them.

4. **Tests are self-describing.** Each `.lox` file declares expectations inline:
   - `// expect: <stdout line>` (per output line)
   - `// expect runtime error: <message>`
   - `// [line N] Error ...` for compile errors
   - `// nontest` excludes a file. Exit codes: 65 compile error, 70 runtime error.
   jlox and clox must produce byte-identical output; error message wording is pinned by tests.

5. **clox debug toggles** live in `c/common.h` (`DEBUG_PRINT_CODE`, `DEBUG_TRACE_EXECUTION`, `DEBUG_STRESS_GC`, `DEBUG_LOG_GC`) — commented out by default; uncomment locally when debugging, don't commit them enabled.

6. **Regenerate `site/` when book-affecting sources change** (`book/`, snippet-bearing code, `asset/`) if the change is meant to be published; the HTML is committed.

## Chapter ↔ code mapping

- jlox: chapters 4–13 (`book/scanning.md` → `inheritance.md`); incremental versions built via `make java_chapters` into `gen/chap04_*` … `gen/chap13_*`.
- clox: chapters 14–30 (`chunks-of-bytecode.md` → `optimization.md`); `gen/chap14_*` … `gen/chap30_*`.
- Appendix I lists the full Lox grammar; Appendix II the generated syntax classes. `note/answers/` has exercise solutions.
