# Prescribe
Executable pseudocode for learning algorithms.
> [!NOTE]
> Work in progress.  
> Built with help from ChatGPT Codex.


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

### Build a runtime-free CLI (bundled Node)
```bash
npm run build:bin
```

Outputs platform binaries to `dist-bin/` (actual names include a platform suffix).

These binaries run without a local Node installation.

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
