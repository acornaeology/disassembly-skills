---
name: 6502-disassembly
description: Method for producing a verified, densely annotated disassembly of 6502 or 65C02 machine code (Acorn, BBC Micro and similar) using the dasmos disassembler, optionally checked with fantasm. Use when disassembling, reverse-engineering or annotating a ROM or disc-loaded program, writing or fixing a dasmos driver script, reassembling to verify a round trip, hunting for mis-decoded code or data, or improving comments, labels and subroutine descriptions in such a project.
---

# 6502 disassembly with dasmos

This skill describes how to turn a raw 6502 binary into an annotated
disassembly that reassembles byte-for-byte to the original, and how to keep
improving it without breaking that guarantee. The method was developed on
Acorn ROMs and disc-loaded programs (NFS, ANFS, ADFS, Econet bridge, Tube
clients, BBC BASIC, drivers) but applies to any 6502 code.

## Principles

1. **The round trip is the ground truth.** The generated assembly must
   reassemble to a byte-identical copy of the original binary. Re-verify
   after every change, however small. A disassembly that does not round-trip
   is broken, whatever it looks like.
2. **The driver script is the source; the listing is generated.** All
   knowledge lives in the dasmos driver (labels, entries, data
   classification, comments, banners, operand expressions). Never hand-edit
   generated `.asm` or `.json`.
3. **Annotate densely and at the right level.** Every instruction gets an
   inline comment that says what it achieves in domain terms, not what the
   mnemonic already says. Every routine and every separate data region gets
   a banner with a title and a description.
4. **Claim only what the evidence shows.** Every statement in a comment must
   be supported by the code, the data, or a cited source. When something is
   an inference, either confirm it (trace it, read the bytes, run it in an
   emulator) or say less.
5. **Describe the program as it is.** Published text (comments, banners,
   docs) describes the binary, never how the disassembly evolved. No old
   label names, no "was previously decoded as", no process narrative. That
   history belongs in version control.
6. **Use the tools to look; don't write ad hoc disassemblers.** To see what
   bytes really are, fix the driver (an entry point, a data declaration) and
   re-render. dasmos is the decoder; fantasm is the query and check layer.

## The working loop

1. **Seed.** Load the binary at its load address. Declare the entry points
   you know: the execution address of a `*RUN` program, the language and
   service entries of a ROM header, vectors the code installs, and the
   targets of dispatch tables once found. See
   [references/workflow.md](references/workflow.md).
2. **Trace and classify.** Render, then read the listing for code that is
   really data (tables, strings, addresses) and data that is really code
   (routines reached only through a dispatch table or vector). Fix each by
   declaring entries or data in the driver. See
   [references/idioms.md](references/idioms.md) for the patterns that cause
   most mis-decodes.
3. **Verify.** Reassemble and byte-compare. Fix any failure before doing
   anything else.
4. **Name and annotate.** Replace auto-generated labels with meaningful
   names, write banners for every routine and data region, comment every
   instruction, and make address operands symbolic. See
   [references/annotation.md](references/annotation.md).
5. **Hunt defects.** Systematically look for the defect classes that slip
   past verification: labels a byte out, comments copied onto the wrong
   code, tables decoded as code, raw addresses in immediates. See
   [references/defects.md](references/defects.md).
6. **Confirm inferences.** Check every behavioural claim you are not sure of
   against the bytes, the documentation, or an emulator. See
   [references/verification.md](references/verification.md).
7. Repeat from 2. Work one routine or region at a time; the full listing of
   even an 8 KB ROM is thousands of lines.

## Tools

- **dasmos**: the programmable disassembler. A Python driver script
  configures a `dasmos.Disassembler`, which renders beebasm (or 64tass)
  assembly and a structured JSON description. Essentials and gotchas:
  [references/dasmos.md](references/dasmos.md). Full guide:
  <https://acornaeology.github.io/dasmos/driver_api.html>.
- **fantasm**: optional but strongly recommended. It runs the driver,
  verifies the round trip, lints annotation addresses, and provides the
  queries and checks used throughout this method (comment consistency, data
  runs, undeclared subroutines, call graphs, assembly extraction). It adapts
  to an existing project layout through a `fantasm.toml`. Setup and the
  commands that matter: [references/fantasm.md](references/fantasm.md).
  Full guide: <https://acornaeology.github.io/fantasm/>.
- **An assembler** for the round trip: beebasm by default.
- **An emulator** with a debugger (memory view, breakpoints, watchpoints)
  for confirming behaviour.

## Starting a new project

Write a `CLAUDE.md` in the project recording the facts this skill cannot
know: the binary files, their load and execution addresses, the CPU, the
build and verify commands, any relocated blocks or overlays, and project
conventions. A template is in
[references/project-claude-md.md](references/project-claude-md.md).

## Working habits

- Re-verify after every batch of driver changes, and before every commit.
- Keep commits focused: one class of fix, or one region, per commit.
- When a fix applies to a pattern, find every instance of the pattern (in
  every version or file) before moving on.
- Build small throwaway scripts for repetitive analysis, but feed their
  conclusions back into the driver; never patch outputs.
- When you find a defect you will not fix now, record it (an issue, a TODO
  list) so it is not lost.
