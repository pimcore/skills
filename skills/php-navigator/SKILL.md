---
name: php-navigator
description: >-
  Fast, token-efficient navigation of PHP codebases using a symbol index
  (ctags) plus ripgrep instead of slow filesystem scans. Use this skill
  WHENEVER working in a PHP project and you need to locate code: finding where
  a class, interface, trait, function, method, constant, or property is DEFINED;
  finding everywhere a symbol is USED; jumping to a definition; or getting an
  outline of a file. Reach for this skill any time you would otherwise run
  `find`, `grep -r`, `ls -R`, or a broad `rg` over a PHP repo to answer "where
  is X?" — even if the user just says "find", "where is", "show me the class",
  "look up", or "go to". Prefer it over raw search tools for any PHP repo of
  non-trivial size.
---

# PHP Navigator

Navigating a PHP codebase with `find` and `grep -r` is slow and wasteful: `find`
only matches filenames, and `grep` scans raw text and returns every mention of a
name, so the one *definition* you want is buried among dozens of usages. Reading
all that noise burns time and context.

This skill replaces that with a **symbol index**. `ctags` precomputes a sorted
map from every class/function/method/etc. to its exact `file:line`, so "where is
`PaymentProcessor` defined?" becomes a single indexed lookup that returns one
line instead of a tree-wide scan. Usage searches still go through `ripgrep`,
which is far faster than `grep -r` and respects `.gitignore`. The net effect is
fewer tool calls, far less output to read, and precise answers.

The helper for all of this is `scripts/phpnav`.

## Setup

The script needs two tools on the machine: `ctags` (Universal Ctags, *not* the
old Exuberant Ctags) and `rg` (ripgrep). Check and install if missing:

```bash
command -v ctags >/dev/null && command -v rg >/dev/null && echo ok
# Debian/Ubuntu:  apt-get install -y universal-ctags ripgrep
# macOS (brew):   brew install universal-ctags ripgrep
```

When this skill is installed as its bundled plugin, a SessionStart hook runs
`phpnav index` automatically at the start of every session, so the index is
normally already built before the first lookup — you do not need to run it by
hand. (If the tools above are missing, the hook skips indexing and prints a note
rather than failing.) Add `.php-tags` to the project's `.gitignore`; it is a
regenerable build artifact, not source. You can still rebuild on demand at any
time with `./scripts/phpnav index`.

## Choosing the right command

The key decision is **definition vs. usage vs. filename** — pick the command
that matches the question, rather than defaulting to a text search:

- "Where is `X` defined / declared?", "go to the class/method", "show me the
  implementation" → **`def`**. This is the common case and the biggest win.
- "Where is `X` used / called?", "what references this?", "find all callers" →
  **`refs`**.
- "What's in this file?", "list the methods of this class" → **`outline`**.
- A genuine free-text or regex search (a string literal, a SQL fragment, a
  config key) that is not a symbol name → **`grep`**.
- Only fall back to `find` when you truly need to match *filenames* (e.g. "list
  every `*Test.php`"). For locating code, `find` is the wrong tool.

## Commands

```bash
./scripts/phpnav def <symbol>      # where a symbol is DEFINED  -> file:line [kind] name
./scripts/phpnav refs <symbol>     # everywhere a symbol is USED (PHP files only)
./scripts/phpnav outline <file>    # symbols defined in one file, in source order
./scripts/phpnav grep <pattern>    # free-text/regex search, restricted to PHP files
./scripts/phpnav index             # rebuild the index
```

`def` may legitimately return more than one line — e.g. a method declared on an
interface and implemented on a class, or several classes sharing a method name.
That is correct and useful; present the candidates rather than assuming the
first is the only one. The `[kind]` tag tells them apart: `c` class, `i`
interface, `t` trait, `f` function/method, `d` constant, `n` namespace, `v`
variable/property.

**Example session:**

```
$ ./scripts/phpnav def PaymentProcessor
src/Payment/PaymentProcessor.php:10  [c]  PaymentProcessor

$ ./scripts/phpnav def refund
src/Payment/PaymentProcessor.php:7   [f]  refund    # interface declaration
src/Payment/PaymentProcessor.php:17  [f]  refund    # class implementation

$ ./scripts/phpnav refs PaymentProcessor
src/checkout.php:6:$processor = new PaymentProcessor();
```

## Keeping the index fresh

The index is a snapshot taken at session start (by the bundled hook) or whenever
you last ran `index`. After you create or substantially edit PHP files in a
session — especially adding or renaming classes/methods — rerun
`./scripts/phpnav index` before relying on `def` again, or a lookup may point at
a stale line or miss a new symbol. `refs`, `outline`, and `grep` always read the
live files, so they never go stale.

## When you need semantic accuracy

`ctags` is fast and dependency-free but purely lexical: it does not resolve
namespaces, inheritance, or `use` aliases, so it cannot reliably answer "which
concrete classes implement this interface?" or follow a fully-qualified name
through imports. When a task needs that level of precision, see
`references/phpactor.md` for using Phpactor's PHP-aware indexer as an upgrade.
