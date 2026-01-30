# Prescribe
Executable pseudocode for learning algorithms.

Source files must use the `.prsd` format (see `docs/prescribe/03-lexical-rules.md`).

## Operations

### Install
```bash
npm install
```

### Build
```bash
npm run build
```

### Run (CLI)
```bash
node dist/cli/main.js <file.prsd>
```

Input is read from stdin as whitespace-delimited tokens. Output is written to stdout.

### Example
```bash
cat <<'EOF' > /tmp/hello.prsd
PRSD 1.0
## Hello
:::prescribe
PROGRAM Hello
    OUTPUT "Hello, " & "PRSD!"
ENDPROGRAM
:::
EOF

node dist/cli/main.js /tmp/hello.prsd
```

### Notes
- Only `.prsd` files are supported by the CLI.
- Code blocks must be fenced with `:::prescribe` / `:::` inside the `.prsd` file.
