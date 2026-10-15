# Defect hunting

A listing can reassemble perfectly and still be wrong: mis-decoded regions,
labels in the wrong place, comments describing different code. These
defects are invisible to the round trip, so look for them deliberately.

## Defect classes that keep recurring

1. **Tables decoded as code.** A data table after an always-taken branch,
   or declared with `d.subroutine`, renders as nonsense instructions
   (`ROR A`, `BRK`, `JSR` to an address outside the program). The giveaway is
   instructions whose comments make no sense, or byte values that are all
   printable characters or all plausible addresses.
2. **Code decoded as data.** A routine reached only through a dispatch
   table or vector, never declared, renders as `EQUB`. Inline comments that
   describe instructions on `EQUB` lines are a strong signal.
3. **Labels and entry points one byte out.** An entry declared at a
   dispatch-table value instead of value + 1, or a label carried from another
   version at an address one byte off. The result is a label inside an
   instruction and a garbled decode either side of it.
4. **Comments copied onto different code.** When annotations are carried
   by address (from a related version, or after code moves), a comment can
   land on a different instruction. The address is still valid, so nothing
   flags it. Check comments whose wording does not fit the instruction:
   branch phrasing on a load, "store" on a compare.
5. **Comments quoting the wrong address.** `Install handler at &9843` on
   an instruction that installs `&989A`. Often a sign the comment came from
   another version. Rendering operands symbolically exposes these.
6. **Misresolved indexed bases.** `LDA unrelated_label,Y` where the
   operand is really `table - &81`. See
   [idioms.md](idioms.md#tables-indexed-by-a-biased-value).
7. **Wrong names.** A routine named for one caller's use, or for a guess
   that turned out wrong ("find station" for a routine that sets a
   directory handle). Read the whole routine and its callers before
   trusting a name.
8. **"Unreachable" code that is reached.** A byte annotated as dead that is
   actually an entry point: an NMI handler installed one byte earlier than
   the label, or the second entry of a `BIT abs` skip.
9. **"Padding" that is read.** Bytes described as unused that a lookup
   reads when no other entry matches.
10. **Stale explanations.** Banners that explain an older, wrong reading
    of the code, or refer to labels that no longer exist.

## Systematic searches

Work through these periodically, across the whole program:

- **Unannotated data.** List every data run with no label or comment; each
  one is either a genuine table that needs a banner and comments, or a
  mis-decode. (`fantasm data runs --unannotated`)
- **Code after terminators.** List code that follows an RTS, JMP or RTI
  with no banner or entry declaration; each is a routine that needs one.
  (`fantasm promote --not-declared`, the AfterTerm column)
- **Undeclared call targets.** Every JSR target should be a declared
  subroutine. (`fantasm audit undeclared`)
- **Raw address immediates.** Pairs of immediate loads that combine to an
  address of something labelled; make them symbolic.
- **Comment consistency.** Comments inside an instruction rather than on
  it, comments whose phrasing does not fit their instruction, comments that
  quote addresses or bytes the operand contradicts, copies of a related
  version's comments onto different code. (`fantasm comments check`)
- **Disassembler warnings.** Treat every dasmos warning (a move boundary
  splitting an instruction, an overlapping classification) as a defect.

## When you find one

- Fix the cause in the driver, re-render, re-verify.
- Search for the same pattern everywhere else: the same idiom in other
  routines, the same copied comment in other versions.
- Re-read the surrounding banners; a mis-decode usually took some wrong
  explanation with it.
