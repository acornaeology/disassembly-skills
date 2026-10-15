# fantasm: checks and queries

fantasm runs the dasmos driver, verifies the round trip, lints annotation
addresses, and answers the questions this method keeps asking. It is
optional, but most of the defect searches in [defects.md](defects.md) are
one command with it. Full guide: <https://acornaeology.github.io/fantasm/>;
for disc-loaded programs see
<https://acornaeology.github.io/fantasm/disc_programs.html>.

## Adopting an existing project

fantasm fits the project's existing layout; nothing needs moving. From
the project root:

```sh
pip install fantasm dasmos      # or: uv add fantasm dasmos
fantasm project init --name myprog --layout explicit \
    --binary orig/PROG \
    --driver tools/dis_prog.py \
    --asm build/prog.asm --json build/prog.json
```

- With no `@ADDR` after the binary, the load and execution addresses are
  read from its `.inf` file (`orig/PROG.inf`). Otherwise give them:
  `--binary orig/PROG@1900`.
- This writes only `fantasm.toml`, with one `[[targets]]` entry. Edit it
  freely; unknown keys are rejected, so typos are caught.
- With a single target, commands need no target argument.
- A program split across several files: one target per file, each with its
  own driver, sharing entry points through named external labels.
- fantasm passes the driver its paths in the environment (`FANTASM_BINARY`,
  `FANTASM_ASM`, `FANTASM_JSON`, ...). A driver that hard-codes its own paths
  also works, as long as `fantasm.toml` says where its outputs land.

### A different assembler

`fantasm verify` uses beebasm unless `fantasm.toml` says otherwise:

```toml
[verify]
command = ["beebasm", "-i", "{asm}", "-o", "{output}"]   # the default
cwd     = "{asm_dirpath}"
output  = "{output}"
```

## The commands this method uses

| Command | Use |
|---|---|
| `fantasm disassemble` | Run the driver. |
| `fantasm verify` | Assemble and byte-compare: the round trip. |
| `fantasm lint` | Every annotation address in the driver lands on a real item. |
| `fantasm comments check` | Comments inside items, copied onto different code, contradicting their operand, or quoting addresses that don't fit. |
| `fantasm coverage` | Proportion of code items with inline comments. |
| `fantasm data runs --unannotated` | Data with no label or comment: banners to write, or mis-decodes. |
| `fantasm data classify` | Data that looks like code, and code that looks like data. |
| `fantasm promote --not-declared` | Code after an RTS/JMP/RTI with no banner (the AfterTerm column). |
| `fantasm audit summary` / `audit detail ADDR` / `audit undeclared` | Per-routine extents, callers, flags; JSR targets with no declaration. |
| `fantasm asm extract START [END]` | Read part of the listing by address or label, with comments. |
| `fantasm cfg ...` | Call-graph queries: roots, leaves, depth. |
| `fantasm labels list` | Labels, including those only ever used as indexed bases. |

Addresses can be written `&1900`, `$1900` or `0x1900`.

Run `fantasm <command> --help` for options. Several commands also compare
related versions of a program (address mapping, carrying annotations
across, cross-version comment checks); those need a `[[versions.entry]]`
graph in `fantasm.toml`, described in the fantasm guide.
