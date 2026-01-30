# Prescribe Interpreter Design (TS, CLI)

Goal: a modular, spec-faithful interpreter for Prescribe (all features), with a CLI front-end now and a browser front-end later.

## High-level architecture

Phases:
1. Front-end: source loading (.prsd), lexing, parsing -> AST with locations.
2. Semantic analysis: name binding, type checking, constant evaluation, access checks.
3. Runtime: interpreter executes AST with stack/heap, standard library, and file I/O.

Core rule: evaluation is left-to-right with no short-circuiting, and all type rules match the spec.

## Module layout (proposed)

```
src/
  cli/
    main.ts              # CLI entry, args, file loading, exit codes
  source/
    source_file.ts       # text + line/col mapping
    prsd.ts              # parse .prsd and extract code blocks
  frontend/
    token.ts             # token kinds, keyword table
    lexer.ts             # ASCII source, escapes, comments, locations
    parser.ts            # recursive descent from EBNF
    ast.ts               # node types for statements/expressions/types
  semantics/
    symbols.ts           # symbol table, scopes, symbol kinds
    types.ts             # type system representations
    resolver.ts          # name binding, access control, overload resolution
    const_eval.ts        # compile-time constants
    type_checker.ts      # full type validation and annotation
  runtime/
    values.ts            # runtime value shapes
    store.ts             # stack frames, lvalue handles, heap
    interpreter.ts       # statement/expression evaluation
    stdlib.ts            # LENGTH, RIGHT, MID, etc.
    file_io.ts           # TEXTFILE/RANDOMFILE behavior
  diagnostics/
    errors.ts            # error classes + formatting
    reporter.ts          # format "<ErrorType> at line N: ..."
  util/
    math.ts              # integer/real range checks, div/mod
    strings.ts           # output formatting, escapes
    dates.ts             # parse/validate Gregorian dates
```

## Source loading

- `.prsd` is the required source format; only `:::prescribe` blocks are executable.
- CLI can accept `--block <n>` later; for now, run all blocks in order (each as its own program).

## Lexer design

- Keywords are case-insensitive. Identifiers preserve case.
- Identifier cannot match a keyword in any casing.
- ASCII-only source; allow Unicode `\u2190` assignment arrow in addition to `<-`.
- Comments: `//` to end of line.
- Tokens include: identifiers, literals, operators, delimiters, keywords.
- Literals:
  - INTEGER/REAL with optional leading `-`.
  - STRING/CHAR with escape processing (`\n`, `\r`, `\t`, `\\`, `\"`, `\'`, `\xNN`).
  - DATE literal: `DATE "YYYY-MM-DD"` must parse or raise SyntaxError.

Locations:
- Store line/column for token start; include in AST nodes for diagnostics.

## Parser design

Recursive descent built from the EBNF. Key nodes:
- Program, Block, Declarations (var/const/type/proc/func/class).
- Statements: assign, if, case, for, while, repeat, call, return, input/output, file ops.
- Expressions: binary/unary, literals, names, calls, array indexing, field access, pointer ops, new.
- Types: built-ins, arrays, records, enums, sets, pointers, file types, class types.

Enforce syntactic constraints early where required (e.g., invalid lvalue forms).

## Type system representation

Types are first-class objects in `semantics/types.ts`:
- Basic: Integer, Real, Boolean, Char, String, Date
- Composite: Array(bounds, elemType), Record(fields), Enum(members), Set(baseEnum)
- Ref-like: Pointer(toType), ClassType(name), TextFile, RandomFile(recordType)

Add helpers:
- `isComparable`, `isNumeric`, `isAssignable`, `isOutputtable`, `isSetCompatible`, etc.

## Name binding and scopes

Scopes:
- Block scopes for program, control-flow blocks, procedures/functions, classes.
- Class scope has access sections (public/private).

Symbols:
- Var, Const, Type, Proc, Func, Class, Field, Method, Constructor, Param, EnumMember.

Binding steps:
1. Predeclare types/classes/procs/funcs in a scope before body checking (allows mutual recursion).
2. Bind declarations, build symbol tables, and annotate AST nodes with symbol references.
3. Enforce access: `PRIVATE` members only accessible within the declaring class.

## Constant evaluation

Constant expressions for `CONSTANT` values:
- Literals, other constants, and operators on constants.
- Evaluate at compile-time using the same semantics as runtime, with strict range checks.

## Type checking

Core checks (non-exhaustive):
- All declarations before statements.
- Assignment types match exactly.
- Operators enforce exact operand types and result types.
- `CALL` used only for procedures.
- `RETURN` rules: functions require `RETURN expr` on all paths (else RuntimeError at runtime).
- `BYREF` args must be lvalues; enforce at check-time.
- `FOR` loop variable read-only in body; enforce on assignment.
- `CASE` branch uniqueness and valid types (INTEGER/CHAR/ENUM/DATE).
- Record and array indexing types are INTEGER and within bounds at runtime.
- `NEW` usage: `NEW ClassName(...)` or `NEW Type` for pointer allocations.
- File operations only on TEXTFILE/RANDOMFILE and correct modes.

## Runtime model

Value shapes (runtime/values.ts):
- Scalar: int (number but range-checked), real (number), boolean, char (string length 1), string, date (YYYY-MM-DD + numeric).
- Composite: arrays (nested arrays), records (field map), sets (bitset or Set of enum ordinals).
- Ref-like: pointer { addr }, class ref { objId }, file handles.
- Null: for pointer/class values.

Store model (runtime/store.ts):
- Stack frames hold locals and parameters (by value or byref alias).
- Heap stores pointer targets and class objects.
- Lvalue handles abstract "place" with get/set (var, array element, record field, pointer deref).

Interpreter (runtime/interpreter.ts):
- Executes AST statements; evaluates expressions left-to-right.
- No short-circuiting for AND/OR; always evaluate both sides.
- Assignments enforce runtime checks: range, null deref, bounds.

## Numeric and string semantics

Integer:
- 32-bit signed range; check overflow after each operation.

Real:
- IEEE-754 double. If result is NaN or infinite -> RuntimeError.
- Underflow/overflow -> RangeError (use explicit checks in util/math.ts).

DIV/MOD:
- Euclidean semantics with `0 <= r < |b|`.

OUTPUT conversion:
- REAL formatted with up to 6 digits after decimal, round half away from zero, trim trailing zeros.

INPUT parsing:
- Token-based, whitespace-delimited.
- Type-specific parsing with errors as specified.

## Files

TEXTFILE:
- Line-oriented, READ/WRITE/APPEND modes.
- READFILE uses INPUT parsing rules after trimming whitespace.
- WRITEFILE uses OUTPUT conversion rules plus newline.

RANDOMFILE:
- Fixed-size records; forbid STRING/SET in records.
- SEEK is 1-based; RangeError if < 1.
- GETRECORD/PUTRECORD operate at current position.

All file errors use FileError with line number.

## Classes and methods

Class metadata:
- Fields with access, methods and constructors, optional base class.
- Default constructor if none declared.

Instantiation:
- `NEW ClassName(...)` allocates object, default-inits fields, runs constructor.
- `SUPER(...)` allowed only in constructors; resolves to base constructor.
- `SUPER.Method(...)` dispatches to base method.

Dispatch:
- Dynamic based on runtime class; method lookup follows inheritance chain.

Access control:
- `PRIVATE` members enforced in resolver/type checker.

## Errors and diagnostics

Errors stop execution immediately with:
`<ErrorType> at line <lineNumber>: <message>`

All interpreter errors throw typed exceptions that include a source location, caught at top-level CLI.

## CLI design

`prescribe <file.prsd>`:
- Loads source, runs pipeline, prints diagnostics to stderr, returns non-zero on error.
- Requires `.prsd` and extracts code blocks.
- For now, no REPL.

## Operations (CLI)

### Install
```bash
npm install
```

### Build
```bash
npm run build
```

### Run
```bash
node dist/cli/main.js <file.prsd>
```

Input is read from stdin as whitespace-delimited tokens. Output is written to stdout.

### Error format
`<ErrorType> at line <lineNumber>: <message>`

## Testing plan (future)

- Lexer: tokens, escapes, keyword rules.
- Parser: statement/expr precedence, EBNF coverage.
- Type checker: mismatches and access rules.
- Runtime: standard library, file I/O, errors.
- Conformance tests from appendix.
