# Verification

There are two kinds of verification: the mechanical round trip, which
proves the listing reproduces the binary, and confirmation of meaning, which
checks that what the comments claim is true.

## The round trip

- Render the listing, assemble it, and compare every byte with the
  original. Any difference, however small, means the listing is wrong.
- Run it after every batch of driver changes and before every commit.
  Annotation-only changes (comments, banners, labels, operand expressions)
  must still round-trip; a failure means an expression evaluates to the
  wrong value, or a declaration changed how bytes are grouped.
- With fantasm: `fantasm disassemble` then `fantasm verify`. Without it, run
  the driver, assemble with beebasm (or the configured assembler) from the
  listing's own directory so relative `INCBIN` paths resolve, and compare
  with `cmp`.
- Automate it in CI so nothing unverified is published.

## Confirming meaning

The round trip says nothing about whether a comment is right. For each
claim that is not obvious from the code itself:

1. **Trace it.** Follow callers and callees; check the flags and registers
   at the point in question; check what really reads a table.
2. **Read the bytes.** When a table's layout or a string's terminator is
   in question, look at the actual bytes in the binary, in every version
   concerned.
3. **Consult documentation.** Technical references, programmer's reference
   manuals, protocol descriptions. Cite what you rely on in the banner or
   an accompanying document.
4. **Run it.** An emulator with a debugger settles behavioural questions
   quickly:
   - set a breakpoint at a routine to see when it runs;
   - set read and write watchpoints on a workspace byte to find every
     routine that uses it;
   - inspect memory after an operation to confirm a structure's contents;
   - type commands at the prompt to confirm parsing behaviour, including
     edge cases the code suggests (unexpected separators, abbreviations).

   When reading memory from BASIC, beware of printing control characters:
   `PRINT CHR$(1)` sends VDU 1, which swallows the next byte of output and
   makes a listing look wrong. Print the values with `PRINT ~?addr` instead.
5. **Say less if you cannot confirm.** "Copied after the name" is better
   than an invented meaning for two bytes.

## People and attributions

Names of programmers, credit strings and authorship claims are easy to get
wrong and embarrassing when wrong. Confirm them against a source
(the program's own credits, a manual, an interview, a contemporary
article). In the listing itself, state only what the binary shows ("a
credit to the programmers") and leave identifications to accompanying
documents where the sources can be cited.
