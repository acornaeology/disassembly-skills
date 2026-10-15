# dasmos essentials

dasmos is driven by a Python script that configures a `Disassembler` and
renders the result. Full guide:
<https://acornaeology.github.io/dasmos/driver_api.html>.

## A minimal driver for a disc-loaded program

```python
import os
from pathlib import Path

import dasmos
from dasmos import Align
from dasmos.expr import sym, lo, hi

# fantasm passes paths in the environment; the defaults let the script run alone.
binary_filepath = os.environ.get("FANTASM_BINARY", "orig/PROG")
asm_filepath = Path(os.environ.get("FANTASM_ASM", "build/prog.asm"))
json_filepath = Path(os.environ.get("FANTASM_JSON", "build/prog.json"))

d = dasmos.Disassembler.create(cpu="6502")          # or "65c02"
d.load(binary_filepath, 0x1900)                     # load address
d.program(load_addr=0x1900, exec_addr=0x1906)       # for the SAVE line
d.use_environment("acorn_mos")                      # OS entry points, vectors
d.use_environment("acorn_model_b_hardware")         # hardware registers

d.subroutine(
    0x1906,
    "start",
    title="Program entry point",
    description="""Entered by *RUN. ...""",
)
d.comment(0x1906, "Disable interrupts while installing", align=Align.INLINE)

ir = d.disassemble()                                # annotations must come before this
asm_filepath.parent.mkdir(parents=True, exist_ok=True)
asm_filepath.write_text(str(ir.render("beebasm")), encoding="utf-8")
json_filepath.write_text(str(ir.render("json")), encoding="utf-8")
```

## The calls that matter

| Call | Purpose |
|---|---|
| `d.load(path, addr)` | Load a binary at its load address. |
| `d.program(load_addr=, exec_addr=, reload_addr=)` | Record file metadata for the rendered SAVE. |
| `d.use_environment(name)` | Add OS, hardware or BASIC symbols. |
| `d.add_move(dest, src, length)` | Code stored at `src` runs at `dest`; returns a `Move` to pass as `move=`. |
| `d.entry(addr, name=None)` | Code starts here; trace from it. |
| `d.label(addr, name, ...)` | Name an address (in or outside the binary). Keyword arguments add a description, length and access for workspace. |
| `d.index_base(addr, name, ...)` | Name a table base used only with indexing. |
| `d.subroutine(addr, name, title=, description=, on_entry=, on_exit=)` | Banner for a **code** routine. Marks it as an entry point. |
| `d.banner(addr, title=, description=)` | Banner for **data** (does not make it code). |
| `d.comment(addr, text, align=Align.INLINE)` | Inline comment on the item at `addr`. Without `align`, the comment is a block placed before the item. |
| `d.byte(addr, n)`, `d.word(addr, n)`, `d.string(addr, n)`, `d.fill(addr, n)` | Declare data. `byte(addr, 4)` groups four bytes on one line; declare one at a time when each needs its own comment. |
| `d.stringz(addr)`, `d.stringcr(addr)` | Zero- or CR-terminated strings. |
| `d.rts_code_ptr(lo_addr, hi_addr)` | A PHA/PHA/RTS dispatch entry (target − 1); also marks the target as code. |
| `d.code_ptr(lo_addr, hi_addr)` | An address-table entry (target as is). |
| `d.hook_subroutine(addr, name, hook)` | Run a hook at each JSR to `addr`, e.g. `dasmos.hooks.stringhi_hook`, `stringhi_skip_hook` or `stringz_hook` for inline data. |
| `d.expr(operand_addr, expr)` | Render an operand as an expression: `lo(sym("handler"))`, `hi(sym("table") - 1)`, `sym("table") - 0x81`. Symbols are checked when the model is built. |
| `d.constant(value, name)` | A named constant for immediates. |
| `d.char_literal(operand_addr)` | Render an immediate as a character. |
| `d.include_binary(addr, length, path)` | Carry a region as `INCBIN` of an external file. |

`d.expr`, `d.char_literal` and the other format calls take the address of
the **operand byte**, which for an immediate instruction is the
instruction's address plus one.

## Gotchas

- **All annotation calls come before `d.disassemble()`**; the model is
  frozen afterwards.
- **A comment at an address inside an instruction** attaches to that
  instruction and reads against the wrong line. Key comments to the first
  byte of the instruction or data item.
- **`d.subroutine` on a table** turns the table into code. Use `d.banner`
  and a label.
- **Annotations of relocated code** use runtime addresses (the move's
  destination).
- **Overlapping `d.load` calls** overwrite each other silently. Load
  separate files to separate addresses, and model overlays with moves.
- **Duplicate labels** between a relocated block and its stored copy break
  assembly; give each an explicit name.
- **Byte order.** `d.word` is little-endian. A big-endian address stays a
  pair of bytes with `hi(...)` / `lo(...)` expressions.
- Treat every dasmos warning as something to fix.
