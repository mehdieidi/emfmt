# emfmt

emfmt formats [Eclipse Emfatic](https://eclipse.dev/emfatic/) (`.emf`) files with consistent indentation and aligned
feature columns. It requires Python 3.10 or newer, with no external dependencies or
Eclipse installation.

## Usage

From the repository root, format all `.emf` files in the current directory and its
subdirectories:

```sh
python emfmt.py .
```

The formatter processes files in sorted path order and writes only those whose
formatting changes. To preview changes or check formatting without writing files,
use `--diff` or `--check`:

```sh
python emfmt.py --diff .
python emfmt.py --check .
```

You can pass individual files or directories. Relative paths are resolved from the
current working directory. Use `--exclude` to skip additional directory names;
the option can be repeated.

```sh
python emfmt.py mde/metamodels/pim
python emfmt.py mde/metamodels/shared/kernel.emf
python emfmt.py --exclude generated --exclude archive .
```

With no path argument, the current script searches one directory above the directory
containing `emfmt.py`, regardless of the working directory. Pass `.` explicitly to
limit discovery to the current directory and its subdirectories.

Directory traversal skips `.git`, `.idea`, `.vscode`, `.venv`, `node_modules`,
`target`, and `vendor` by default. Symbolic links are skipped during traversal and
rejected when passed explicitly. An explicitly selected file is processed even if
it is inside an excluded directory.

## Formatting rules

The formatter uses literal tabs for indentation, displayed at four-space tab stops.
Spaces separate words and pad columns within declarations, annotations, and
expressions. The main rules are:

- One tab per nesting level, with opening braces on the declaration line.
- One declaration per line, compact empty blocks, a final newline, and LF layout
  outside preserved comments and strings.
- One blank line between classifiers, between package, import, and declaration
  sections, and when feature kinds change within a class. Consecutive imports
  stay together.
- Existing blank lines inside classes and interfaces separate alignment sections,
  even when both sections contain the same feature kind. Repeated blank lines
  become one. Leading comments and annotations stay with their section.
- A single space between modifiers, the feature keyword, and its type.
  Multiplicity and an optional `#opposite` attach directly to the type, as in
  `ref Item[*]#parent`.
- Feature names aligned within each section, starting one space after the longest
  complete prefix. The prefix includes modifiers, keyword, type, multiplicity,
  and opposite. Shorter prefixes receive spaces to align their names. Default
  assignments align one space after the longest feature name.
- Consistent punctuation spacing, including in annotations and operation parameters.

Section boundaries and the order of declarations, modifiers, annotations, and
imports are preserved.

The following example shows tabs expanded to four-space stops:

```emfatic
class Item {
    attr String[1] ~id;
    attr String    name;
    attr Boolean   enabled = true;

    val Item[*]#parent children;

    ref Item[?]#children parent;
}
```

## VS Code settings

Add these Emfatic language settings to `.vscode/settings.json` to use four-space
tab stops and prevent indentation detection from choosing two-space tabs based
on an older file. These settings are needed when an EditorConfig extension is
not installed.

```json
{
  "[emfatic]": {
    "editor.tabSize": 4,
    "editor.insertSpaces": false,
    "editor.detectIndentation": false
  }
}
```

If an open Emfatic file still shows a tab size of two, reopen it. Run the formatter
to replace any existing space indentation.

## Supported syntax and file safety

The formatter follows the [Emfatic syntax reference](https://eclipse.dev/emfatic/).
It recognizes declarations, nested packages, classes and interfaces, enums,
annotations, and balanced type and parameter expressions. Unfamiliar feature
syntax uses the general layout without column alignment. Features containing
internal comments or multiline literals also use the general layout. The formatter
does not compile or semantically validate models.

Strings and comments retain their exact contents, including internal whitespace
and line endings. Files must decode as UTF-8. UTF-8 byte order marks (BOMs) are
preserved.

Before writing, the formatter compares every token, including comments and strings,
against the original input. It prepares all files before writing any changes.
An incomplete declaration, unbalanced delimiter, encoding error, or token mismatch
aborts the run without changing files.

Each changed file is replaced atomically. The batch is not transactional, so an
I/O failure during replacement can leave earlier files formatted.

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Formatting succeeded, or all files were already formatted. |
| `1` | `--check` or `--diff` found files that need formatting. |
| `2` | An input or filesystem error occurred. |
