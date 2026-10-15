# Annotation standards

The aim is a listing a reader can follow without the binary or the
original source: every routine explained, every instruction commented,
every address symbolic.

## Inline comments

- **Comment every instruction.** Coverage is a goal, not a nicety.
- **Say what it achieves, not what it is.** `Advance to the next FCB slot`,
  not `Increment X`. A comment that only restates the mnemonic adds nothing.
- **Use the domain's terms**: file handles, stations, the URD, the
  receive buffer, not registers and offsets alone.
- **Branches state their condition and its meaning**: `Zero: no files
  open`, `C set: name too long`, `Always taken (V set by the poll)`.
- **Keep them short.** One line, about 60 characters. Longer explanations
  go in the banner.
- **Name reason codes only when they add information**: `OSBYTE &7A: scan
  the keyboard` is useful because `&7A` alone is opaque.
- **Data gets comments too**: each table entry, each field of a template,
  each string's role (`Inline text, printed by print_inline`).

## Banners

Every routine, and every data region that is logically separate from the
code before it, gets a banner: a title and a description.

- **Code:** `d.subroutine(addr, name, title=..., description=...,
  on_entry=..., on_exit=...)`. The title is a standalone phrase ("Copy the
  handles into the parameter block"). The description is complete
  sentences: what the routine does, how it is reached, what it leaves
  behind, and anything surprising.
- **Data:** `d.banner(addr, title=..., description=...)` plus a
  `d.label(addr, name)`. Do **not** use `d.subroutine` on data: it marks the
  address as code, and the disassembler will decode the table as
  instructions.
- **Calling conventions** go in `on_entry` / `on_exit`: registers, flags and
  memory the caller supplies or receives. Record them only when the code
  shows them.
- **Code reached only by branches or tail jumps** after an RTS, JMP or RTI
  still deserves a banner; to a reader it is a separate routine.
- Explain tricks in the banner: shared bytes, biased tables, terminators
  borrowed from the next opcode, data placed after an always-taken branch.

## Labels

- Replace every auto-generated label (`l1234`, `sub_c1234`, `loop_c1234`)
  with a meaningful name.
- Routines: verb phrases (`set_csd_handle`, `print_station_addr`). Data:
  nouns (`boot_cmd_offsets`, `fs_vector_table`). Loop and exit points:
  `loop_...`, `done_...`, `return_...`.
- **A name must not lie.** If a routine does more or less than its name
  says, rename it. Misleading names cause wrong comments elsewhere.
- Workspace and zero-page variables get names, descriptions, sizes and
  access types as their purpose becomes clear.
- For a table reached only as an indexed base, label the table, not the
  byte the instruction's operand happens to point at.

## Operands

- **Addresses are symbolic.** `LDA #<(handler)`, not `LDA #&EF`. Use
  `d.expr` with the `lo`/`hi`/`sym` expression helpers.
- **Constants are named** (`d.constant`) when they have meaning: OSBYTE
  numbers, hardware register bits, protocol values.
- **Character immediates render as characters** (`CMP #'.'`) only when the
  byte really is a character being compared or stored. Offsets, masks,
  control-register values and error numbers stay numeric, even if they
  happen to be printable.

## Prose

- **Describe the program as it is.** No history of the disassembly: no old
  names, no "previously decoded as", no "renamed from". That belongs in
  version control.
- **One version per listing.** When several versions of a program are
  disassembled, keep comparisons out of the per-version listings; put them
  in a separate changes document.
- **Own the claim.** State findings plainly. Where an external source
  informs a name or reading, confirm it against the code and present it in
  your own words.
- **Verify names of people** (credits, authors) against a source before
  writing them anywhere. Never from memory.
- **Hex as `&XXXX`** in prose and comments (Acorn style), `0xXXXX` in
  Python.
