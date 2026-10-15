# Workflow: from raw binary to verified listing

## Know the binary before you start

Record in the project's `CLAUDE.md`:

- **Where it runs.** For a sideways ROM, `&8000`. For a disc-loaded program,
  the load address from its `.inf` file or catalogue entry (`*INFO`), plus
  the execution address.
- **The CPU.** NMOS 6502 for BBC Model B era code; 65C02 for Master code
  that uses CMOS-only instructions (`STZ`, `BRA`, `TSB`, `TRB`, `(zp)`
  addressing). Disassembling 65C02 code as NMOS turns those opcodes into
  junk.
- **Size and checksums**, so you can tell versions apart.
- **What else it needs at runtime**: OS entry points (OSWRCH, OSBYTE...),
  hardware registers, workspace. dasmos environments supply many of these
  symbols (`acorn_mos`, `acorn_model_b_hardware`, `acorn_master_hardware`,
  `acorn_sideways_rom`, `acorn_fdc_8271`, `acorn_fdc_1770`, `bbc_basic_6502`).

## Seeding entry points

dasmos traces code from the entry points you declare and treats everything
it cannot reach as data. Start with what is certain:

- **ROM:** the language entry (`&8000`) and service entry (`&8003`). The ROM
  header (type byte, copyright offset, title, version) is data.
- **Disc-loaded program:** the execution address. Record load, execution
  and reload addresses with `d.program(...)` so the rendered listing saves
  the file with the right metadata.
- **Vectors and handlers the code installs.** When you see code storing an
  address into a vector (`LDA #lo : STA vec : LDA #hi : STA vec+1`), the
  stored address is an entry point.
- **Dispatch-table targets.** Once a table is identified, every target is an
  entry point (`d.rts_code_ptr` / `d.code_ptr` declare the table and its
  targets in one step).
- **Code reached only by JMP from relocated or copied code.**

Then iterate: render, look at the large data regions, and ask whether each
is genuinely data or unreached code. A long `EQUB` run whose bytes decode to
plausible instructions, or whose inline comments describe instructions, is a
candidate for an `entry`.

## Relocated code

Much 6502 code is stored at one address and copied elsewhere to run: Tube
host code copied into pages 4 to 6, NMI handlers copied into workspace, a
driver stored inside a loader and moved down to its runtime address.

- Declare each block with `d.add_move(dest_runtime, src_binary, length)`.
- Annotate relocated code at its **runtime** address, which is what the
  code itself refers to.
- Check the block boundaries carefully. If a move is a byte too long or too
  short, an instruction can straddle the boundary, and dasmos warns about
  it. Treat those warnings as defects.
- Labels generated in both the stored copy and the relocated copy can
  collide; give them explicit, distinct names.

## Disc-loaded and multi-file programs

- The whole file must reassemble, so every byte must be accounted for as
  code or data. Padding, workspace images and embedded data (BASIC
  programs, fonts, tables) all count.
- Read load and execution addresses from the `.inf` sidecar if there is
  one; otherwise from the disc catalogue.
- Embedded content in another format, such as a tokenised BASIC program,
  can be carried as an included binary (`d.include_binary`) regenerated from
  a readable source file, so the listing stays readable and still round-trips.
- A program split across several files is simplest as **one disassembly
  per file**, each declaring the other files' entry points and shared
  workspace as named external labels. Overlays (several files loaded into
  the same window at different times) are a second reason to keep them
  separate.

## Deriving a disassembly for a related version

When a second version of a program differs only slightly from one already
annotated, map addresses between the two and carry annotations across rather
than starting again.

- Map by aligning opcode sequences; then recover routines the alignment
  missed by following JSR targets, anchoring on strings, decoding dispatch
  tables and fingerprinting opcodes.
- Afterwards, check **structure**, not just coverage. Address mapping
  carries comments well but silently drops structural declarations: dispatch
  tables, command tables, data classifications and operand expressions. Diff
  the parent and child drivers for these.
- Check every carried comment against the instruction it now sits on. The
  commonest defect in a derived version is a comment copied by address onto
  different code, or a description quoting the parent's addresses.

## The loop in practice

- Make one kind of change at a time, re-render, re-verify.
- Read the listing in small pieces (a routine, a table), not end to end.
- Prefer finding a pattern and fixing all its instances over fixing one
  site at a time.
