# 6502 idioms that cause mis-decodes

Most disassembly errors come from a handful of idioms that make code look
like data or data look like code. Recognise them, then model them
explicitly in the driver.

## Dispatch through the stack (PHA/PHA/RTS)

```
LDA table_hi,X : PHA
LDA table_lo,X : PHA
RTS                      ; "returns" to the pushed address + 1
```

- The tables hold **target minus 1**, because RTS adds one.
- Low and high bytes are often in separate tables, sometimes far apart.
- When every handler lies in one page, there may be only a low-byte table
  and an immediate high byte (`LDA #&84 : PHA`).
- Model with `d.rts_code_ptr(lo_addr, hi_addr)` per entry, which both
  renders `<(target - 1)` / `>(target - 1)` and marks each target as code.
- The same trick pushes a **return address** before dispatching, so the
  handler's RTS comes back to a loop: `LDA #>(loop - 1) : PHA :
  LDA #<(loop - 1) : PHA`. Render those immediates symbolically too.

## Indirect jumps and vectors

`JMP (vector)` and tables of full addresses (`.word handler`) are the other
dispatch forms. Use `d.code_ptr` for address tables, and find out what
installs each vector to learn its handlers.

## Inline data after a JSR

A print routine that pops its return address and prints the bytes that
follow the JSR, then returns past them:

```
JSR print_inline
EQUS "Bad command"      ; not code
...next instruction
```

- Without help, the disassembler decodes the string as instructions.
- Use a hook (`d.hook_subroutine` with `dasmos.hooks.stringhi_hook`,
  `stringhi_skip_hook` or `stringz_hook`, or a custom one) to consume the
  data and resume decoding after it.
- Check how the string ends. Common terminators: a zero byte, a CR, or
  **the next byte with bit 7 set**, which is often simply the opcode of the
  following instruction (`NOP`, `LDA #`, `JSR` and many others have bit 7
  set). The string's last character and the next opcode can be adjacent
  with no separator at all.
- Error routines often take an error number and inline text the same way.

## Tables indexed by a biased value

When code indexes a table by a value that starts above zero, the base in
the instruction is the table address minus that bias:

```
LDY control_byte          ; &81..&88
LDA table - &81,Y         ; operand points &81 bytes before the table
```

The disassembler resolves the operand to whatever label sits at
`table - &81`, often an unrelated routine. Render it as an expression
(`d.expr(operand_addr, sym("table") - 0x81)`) and label the table itself.
The same applies to tables read from base+1 or base+2 (`table+1,X` for the
second byte of each entry).

## Data after a branch that is always taken

```
LDA #&80                  ; A is non-zero
BNE somewhere             ; always taken
EQUS "L.!BOOT"            ; never executed
```

A disassembler treats every conditional branch as possibly falling through,
so it decodes the bytes after an always-taken branch as code. Look for
branches whose condition is fixed by the previous instruction: `BNE` after
loading a non-zero value, `BVS` after `BIT` of a byte known to have bit 6
set, `BCS` after `SEC`, `CLV : BVC`, `BPL` after a loop that only exits one
way. Declare the following bytes as data, and comment the branch as
"always taken" with the reason.

## Addresses as immediates

Code builds pointers from immediate bytes:

```
LDX #&40 : LDY #&8D : JSR oscli      ; command string at &8D40
LDA #&EF : LDY #&81 : JSR set_vector ; next handler at &81EF
```

These are addresses, not constants. Render them as `<(label)` / `>(label)`
with `d.expr(operand_addr, lo(sym("label")))` and `hi(...)`. Doing so
documents the link, keeps it correct if the code is reassembled elsewhere,
and frequently exposes comments that quote the wrong address.

## High byte first, as a terminator

Some tables store an address big-endian, against the 6502's usual order,
because the high byte then has bit 7 set (for code in `&8000`–`&FFFF`) and
can terminate the preceding name:

```
EQUS "BYE"
EQUB >(bye_handler - 1), <(bye_handler - 1)
```

A plain `EQUW` would emit the bytes in the wrong order; keep the explicit
byte pair and say why in a comment.

## Overlapping and shared bytes

Space-saving tricks share bytes between items:

- A string whose tail is another string: `"!BOOT"` is the end of
  `"L.!BOOT"`, and a command table points at both.
- A terminator that doubles as the next item: the CR ending one command
  is, on its own, an empty command.
- A table base placed inside an earlier instruction or string, with the
  index range chosen so only the intended bytes are read.
- A comparison loop that runs one entry past its table and compares
  against the following opcode.

Declare the items at their real boundaries, label each entry point into the
shared bytes, and explain the sharing in the banner.

## Skipped bytes

`BIT abs` (`&2C`) is used to skip the next two bytes: a second entry point
lands in its operand. Code jumping into the middle of an instruction is
rare otherwise; when a label lands inside an instruction, suspect a
mis-decode first, then this idiom.

## Self-modifying code

Code that writes into its own operands (an `STA` whose target is inside a
later instruction) is common in fast loops and relocated code. Label the
modified operand, comment both the write and the modified instruction, and
remember the listing shows the initial value only.

## Copy loops

A loop copying a block (`LDA src,Y : STA dest,Y : DEY : BNE`) is usually
relocation or template installation. Identify what is copied and model it
(an `add_move` for code, labelled data for templates). The source and
destination operands are index bases, not variables.
