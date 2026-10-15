# Template: CLAUDE.md for a disassembly project

Copy this into the project's `CLAUDE.md` and fill it in. It records what the
6502-disassembly skill cannot know about a particular project.

```markdown
# CLAUDE.md

## What this is

Annotated disassembly of <PROGRAM> (<machine>, <year>), produced with dasmos
and verified by reassembly. Follow the 6502-disassembly skill's method.

## Binaries

| File | Load | Exec | Size | Notes |
|---|---|---|---|---|
| orig/PROG | &1900 | &1906 | &1A80 | main program |
| orig/PROG2 | &6800 | &6800 | &0400 | overlay, loaded at run time |

- CPU: 6502 (NMOS) | 65C02
- Relocated blocks: stored &xxxx–&xxxx, runs at &yyyy
- Embedded non-code content: <tokenised BASIC, fonts, ...> and how it is carried

## Commands

```sh
fantasm disassemble [TARGET]   # run the driver(s)
fantasm verify [TARGET]        # must report a byte-identical match
fantasm lint [TARGET]
fantasm comments check [TARGET]
```

(or the equivalent manual steps: run the driver, assemble with beebasm from
the listing's directory, `cmp` against the original)

## Layout

- Drivers: tools/...
- Generated listings: build/... (never edit by hand)
- Notes and analyses: docs/...

## Conventions

- Hex as &XXXX in prose and comments, 0xXXXX in Python.
- Comment every instruction; banner every routine and data region.
- Published text describes the program as it is; no history of the
  disassembly in comments, banners or docs.
- <project-specific naming, workspace names, terminology>

## Known facts and open questions

- <confirmed facts worth keeping in mind, with sources>
- <open questions to check in an emulator or against documentation>
```
